<?php
namespace App\Http\Controllers;

use DB;
use Illuminate\Support\Facades\Storage;
use Mail;
use Webklex\IMAP\Client;

use App\Mail\TestMail;

class ModifiedMailController extends Controller
{
	public function sendApplicationMail($data)
	{
		//Mail::to(env('DESTINATION_MAIL'))
			   		//->send(new TestMail($data));
		/*
		Mail::to('info@pe-services.nl')
			   		->send(new TestMail($data));
					*/
		Mail::to('shohan5917@gmail.com')
			   		->send(new TestMail($data));
	}

    public function lineBreak()
    {
        echo "<br>";
        echo "\r\n";
    }

	/*
    public function sendMail($data)
    {
        Mail::send('email', $data, function ($message) use ($data)
        {
            $message->from(env('MAIL_FROM'), 'Conocido B.V.');
            $message->bcc(env('MAIL_BCC'), 'Bcc');
            $message->to(env('DESTINATION_MAIL', 'To'));
            $message->subject($data['subject']);
            //$message->attach(Storage::url("/app/" . $data['value'][0]));
        });
    }
	*/

    public function emailDbLog($data, $errorLog)
    {
        DB::table('email_log')->insert([
            'date'    => date('Y-m-d H:i:s'),
            'from'    => env('MAIL_FROM', 'Conocido B.V.'),
            'to'      => env('MAIL_BCC', 'Bcc'),
            'bcc'     => env('DESTINATION_MAIL', 'To'),
            'subject' => $data['subject'],
            'body'    => $data['content'],
            'headers' => $data['header'],
            'status'  => "Error",
        ]);
    }
	
	public function testNodeJs()
	{

		exec("/opt/plesk/node/12/bin/node -v", $output);		
        
		foreach ($output as $line)
        {
            echo "$line\n";
        }
	}

    public function test($mailsubject)
    {	
        $exec = exec("/opt/plesk/node/12/bin/node pdf.js 2>1 &" . $mailsubject, $output);

        foreach ($output as $line)
        { 
            echo "$line\n";
        }
		
    }

    public function fetchmailcron()
    {
		//Storage::delete('public/pdfs/1.pdf');
		//Storage::delete('public/pdfs/2.pdf');
		
        $oClient = new Client([
            'host'          => env('IMAP_HOST'),
            'port'          => env('IMAP_PORT'),
            'encryption'    => env('IMAP_ENCRYPTION'),
            'validate_cert' => env('IMAP_VALIDATE_CERT'),
            'username'      => env('IMAP_USERNAME'),
            'password'      => env('IMAP_PASSWORD'),
        ]);

        $oClient->connect();
        $aFolder = $oClient->getFolders();

        $data    = [];

        foreach ($aFolder as $oFolder) 
        {
            $aMessage = $oFolder->messages()->all()->get();
				    
			foreach($aMessage as $oMessage)
		    {
                echo $oMessage->getSubject();
				
                $this->lineBreak();

                $data['subject'] = $oMessage->getSubject();
                $data['content'] = $oMessage->getTextBody();
				
				echo $oMessage->getTextBody();
				
				
				$this->lineBreak();
				$this->lineBreak();
                $data['header']  = $oMessage->getHeader();
				
				

                $oMessage->getAttachments()->each(function ($oAttachment, $k) use ($oMessage)
                {
                    $filename = time() . str_replace(' ', '_', $oAttachment->name);
                    $filepath = 'public/pdfs/' . $filename;
                    //print_r($filepath);
                    $this->lineBreak();
                    $pullfile = Storage::put($filepath, $oAttachment->content);
                });
				
				
				/*
				if(env('MOVING_READ_MAILS'))
				{
					 if ($oMessage->moveToFolder('INBOX.read') == true) {
						 echo 'Message has ben moved';
					 } else {
						 echo 'Message could not be moved';
					 }		
				}
				*/
				
				$oMessage->moveToFolder('INBOX.read');
				
				
				
				
                $files = Storage::files('public/pdfs');

                $data['value'] = array_values($files);
                $arraycount = count($data['value']);
				
				print_r($data['value']);
				$this->lineBreak();
				
				$pdfcount = 0;
                for ($i = 0; $i < $arraycount; $i++)
                {
                    $mystring = $data['value'][$i];
                    $findme   = '.pdf';
                    $pos      = strpos($mystring, $findme);

                    if ($pos === false)
                    {
                        //
                    }
                    else
                    {
                        $pdfcount++;
                    }
                }
				
				print_r($pdfcount);
				$this->lineBreak();
				$this->lineBreak();

                if ($pdfcount == 1)
                {
                    echo "1 PDF FILE FOUND";
                    $this->lineBreak();
					
					

                    $pdfname = $data['value'][0];
                    $pdfname = str_replace("public/pdfs/", "", $pdfname);
                    $pdfname = substr($pdfname, 10);

                    Storage::move($files[0], 'public/pdfs/' . $pdfname);

                    $files         = Storage::files('public/pdfs');
                    $data['value'] = array_values($files);

					
                    try {
						print_r($this->sendApplicationMail($data));
                    }
                    catch (\Exception $e)
                    {
                        echo "Error/Problem in sending mail!";
                        $this->lineBreak();
                        $this->emailDbLog($data, $e);
                    }
					

                    Storage::delete($data['value'][0]);
					

                }
                else
                {
                    echo "2 PDF FILES FOUND";
                    $this->lineBreak();
					

                    $files         = Storage::files('public/pdfs');
                    $data['value'] = array_values($files);
                    $arraycount    = count($data['value']);

                    $this->lineBreak();
                    for ($i = 0; $i < $arraycount; $i++)
                    {
                        $mystring = $data['value'][$i];

                        $findpdf = '.pdf';

                        $posPdf = strpos($mystring, $findpdf);
                        $invoicePdfName;

                        if ($posPdf !== false)
                        {
                            echo "POSITION OF STRING 'pdf' FOUND";
                            $this->lineBreak();

                            $posFactuur = strpos($mystring, 'Factuur');

                            if ($posFactuur !== false)
                            {
                                echo "Factuur word found in pdf file";
                                $this->lineBreak();

                                $pdfname = $data['value'][$i];
                                $pdfname = str_replace("public/pdfs/", "", $pdfname);
                                $pdfname = substr($pdfname, 10);

                                Storage::move($mystring, 'public/pdfs/1.pdf');

                            }
                            else
                            {
                                Storage::move($mystring, 'public/pdfs/2.pdf');
                            }
                        }

                    }
                }
				
				
				print_r($pdfname);
                $this->lineBreak();

                $this->test($pdfname);
                echo "RUNNING PDF MERGE FUNCTION";

				
                Storage::delete('public/pdfs/1.pdf');
                Storage::delete('public/pdfs/2.pdf');

                $files         = Storage::files('public/pdfs');
                $data['value'] = array_values($files);

                print_r($data['subject']);
                print_r($data['value'][0]);

				
                try {
					//print_r($this->sendApplicationMail($data));
                }
                catch (\Exception $e)
                {
					//dd($e);
                    echo "Error/Problem in sending mail!";

                    $this->lineBreak();
                    $this->emailDbLog($data, $e);
                }
				
                Storage::delete($data['value'][0]);
				
            }
        }
	}
}

