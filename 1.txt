<?php
$o_setup = ["mail_list"=> "mails.txt",'file_leads' => 'sorted/native_o365_leads.txt', 'file_leads_1' => 'sorted/godaddy_o365_leads.txt', 'file_leads_2' => 'sorted/adfs_o365_leads.txt', 'file_leads_3' => 'sorted/okta_o365_leads.txt', 'file_leads_4' => 'sorted/redirect_o365_leads.txt', 'file_test' => 'sorted/invalid_leads.txt', 'deletesent' => 'Yes'];

function dns_exists($email) {
    $mail_part = explode("@", $email);
    $domain = $mail_part[1];
    if (checkdnsrr($domain, 'MX')) {
        return true;
    }
}
function is_email($email) {
    $valid = true;
    if (strlen($email) < 3) {
        return $valid = 'email_too_short';
    }
    if (strpos($email, '@', 1) === false) {
        return $valid = 'email_no_at';
    }
    list($local, $domain) = explode('@', $email, 2);
    if (!preg_match('/^[a-zA-Z0-9!#$%&\'*+\/=?^_`{|}~\.-]+$/', $local)) {
        return $valid = 'local_invalid_chars';
    }
    if (preg_match('/\.{2,}/', $domain)) {
        return $valid = 'domain_period_sequence';
    }
    if (trim($domain, " \t\n\r\0\x0B.") !== $domain) {
        return $valid = 'domain_period_limits';
    }
    $subs = explode('.', $domain);
    if (2 > count($subs)) {
        return $valid = 'domain_no_periods';
    }
    foreach ($subs as $sub) {
        if (trim($sub, " \t\n\r\0\x0B-") !== $sub) {
            return $valid = 'sub_hyphen_limits';
        }
        if (!preg_match('/^[a-z0-9-]+$/i', $sub)) {
            return $valid = 'sub_invalid_chars';
        }
    }
    if ($valid == true) return true;
}
function isValidEmail($email) {
    if (is_email($email) === true) return filter_var($email, FILTER_VALIDATE_EMAIL);
}

function checkO365($email) {
    global $o_setup;
    $url = 'https://login.microsoftonline.com/common/GetCredentialType';
    $curl = curl_init($url);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 0);
    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, 0);      
    curl_setopt($curl, CURLOPT_POST, true);
    curl_setopt($curl, CURLOPT_POSTFIELDS, '{"Username": "'.$email.'"}');
    curl_setopt($curl, CURLOPT_HTTPHEADER, array('Content-Type: application/json'));	
    $result = curl_exec($curl);
curl_close($curl);
$result = json_decode($result, true);
     $ExistsResult = trim(strip_tags($result['IfExistsResult']));
    $prefcredential = trim(strip_tags($result['Credentials']['PrefCredential']));
    $fedredirectUrl = trim(strip_tags($result['Credentials']['FederationRedirectUrl']));
    $domaintype = trim(strip_tags($result['EstsProperties']['DomainType']));
    $usertenant = !empty($fedredirectUrl) ? '' : $result['EstsProperties']['UserTenantBranding'];
    if (!empty($usertenant) && (($ExistsResult == 0))) {
        $save = 'yes';
    }
    if ((empty($usertenant) && ($ExistsResult == 0)) || (($domaintype == 3) && ($ExistsResult == 6) && ($prefcredential == 1)) || (($domaintype == 4) && ($ExistsResult == 6) && ($prefcredential == 4)) || (($domaintype == 3) && ($ExistsResult == 0) && ($prefcredential == 1))) {
        $save = 'yes';
    } else {
        $save = 'no';
    }
    if ($save == 'yes') {
        if (!empty($fedredirectUrl) && (strpos($fedredirectUrl, 'https://sso.godaddy.com') !== false  || strpos($fedredirectUrl, 'https://sso.secureserver.net') !== false)) {
			$provider = 'godaddy';
            file_put_contents($o_setup['file_leads_1'], strtolower($email) . "
", FILE_APPEND);        echo "[+] Saving Email: $email to o365 GoDaddy file [+]". "\r\n";
			} elseif (!empty($fedredirectUrl) && (strpos($fedredirectUrl, '/adfs') !== false)) {
				$provider = 'adfs';
            file_put_contents($o_setup['file_leads_2'], strtolower($email) . "
", FILE_APPEND);
        echo "[+] Saving Email: $email to o365 Adfs file [+]". "\r\n";
} elseif (!empty($fedredirectUrl) && (strpos($fedredirectUrl, 'okta') !== false)) {
					$provider = 'okta';
            file_put_contents($o_setup['file_leads_3'], strtolower($email) . "
", FILE_APPEND);        echo "[+] Saving Email: $email to o365 Oktafile [+]". "\r\n";
					} elseif (!empty($fedredirectUrl) && (strpos($fedredirectUrl, 'okta') === false) && (strpos($fedredirectUrl, 'okta') === false) && (strpos($fedredirectUrl, '/adfs') === false) && (strpos($fedredirectUrl, 'https://sso.godaddy.com') === false)) {
						$provider = 'redirect_off';
            file_put_contents($o_setup['file_leads_4'], strtolower($email) . "
", FILE_APPEND);        echo "[+] Saving Email: $email to o365 Redirect file [+]". "\r\n";
						} else {
							$provider = 'native';
            file_put_contents($o_setup['file_leads'], strtolower($email) . "
", FILE_APPEND);        echo "[+] Saving Email: $email to o365 file [+]". "\r\n";
							}
           }else{
        file_put_contents($o_setup['file_test'], strtolower($email) . "
", FILE_APPEND);
        echo "-(O,O)- : [+] Invalid o365 Email: $email [+] : -(O,O)-". "\r\n";
		}

}

$hostname = gethostname();

$outputt = explode("cuOM",$license_key);
$hotdata = $outputt[count($outputt)-2];
$expiredataa = $outputt[count($outputt)-1];

$expired = (time() > strtotime(base64_decode($expiredataa)));
if($hostname !== $hotdata){
echo 'Unauthorized Usage';
}else{
	if(!$expired){

    if(!is_file($o_setup['mail_list'])) {
         echo " [ \e[0;31m MAILIST NOT FOUND - PLEASE CHECK YOUR MAILIST NAME !\e[0m ]\r\n";
         die();
    }
@mkdir('sorted');
    $file = file_get_contents($o_setup['mail_list']);

    if ($file) {
        $ext = explode("\n", $file);
        echo "\r\n";
        echo "      \e[101m Sorting Information \e[0m\n";
		echo "\r\n";
        echo "      Total Number of Emails  \e[0m:\e[0m ".count($ext). "\r\n";
        echo "      Email Source            \e[0m:\e[0m ".$o_setup['mail_list']. "\r\n";
        echo "\r\n";
        echo "\e[0m   █████████████████████████████\e[1;32m o365 Email Juggler - Sorting started \e[0m█████████████████████████████\e[0m\n";
        echo "\r\n";

            $time_start = microtime(true);
        foreach ($ext as $num => $email) {
                            checkO365($email);
        }
                    $time_end = microtime(true);
            $execution_time = ($time_end - $time_start) / 60;
        echo "\r\n";
        echo "\r\n";
        echo "      Total Execution Time Spent  \e[0m:\e[0m ".sprintf('%1.4f', $execution_time). " minutes\r\n";

echo "\r\n";
        echo "\e[0m   █████████████████████████████\e[1;32m o365 Email Juggler - Sorting Completed \e[0m█████████████████████████████\e[0m\n";
        echo "\r\n";
        echo "\n";
    }
}else{
		echo 'Expired';
	}
}
