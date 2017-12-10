# bose-soundtouch-controller-php
taken from http://www.nikolaus-lueneburg.de/2017/01/bose-soundtouch-php-gateway-teil-1/
see also http://www.nikolaus-lueneburg.de/2017/02/bose-soundtouch-php-gateway-teil-2/

<html>
<head>
	<meta charset="utf-8" />
	<title>BOSE SOUNDTOUCH</title>
</head>

<body>

<?php

$device = $_GET['device'];
$action = $_GET['action'];
$value = $_GET['value'];

if (!is_numeric($device) || empty($device)) 
{
	die ('Wrong or missing parameter Device');
}

//////////////////////////////////////////
// Variablen
$DeviceIP = "192.168.1.". $device;

//////////////////////////////////////////
// Functions

    // Press Key
    function PressKey($key)
    {
		global $DeviceIP;

        $xmldata1 = "<key state=press sender=Gabbo>". $key ."</key>";
        $xmldata2 = "<key state=release sender=Gabbo>". $key ."</key>";
        for ($i=1 ; $i < 3 ; $i++) {
            $curl = curl_init();
            curl_setopt_array($curl, array(
                CURLOPT_URL => "http://". $DeviceIP .":8090/key",
                CURLOPT_HEADER => false,
                CURLOPT_RETURNTRANSFER => true,
                CURLOPT_POST => true,
                CURLOPT_POSTFIELDS => ${"xmldata".$i},
                CURLOPT_HTTPHEADER => array("Content-type: text/xml")));
            $result = curl_exec($curl);
            curl_close($curl);
        }
    }

    // Status
    function Status()
    {
		global $DeviceIP;

        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL => "http://". $DeviceIP .":8090/now_playing",
            CURLOPT_HEADER => false,
            CURLOPT_RETURNTRANSFER => true));
        $result = curl_exec($curl);
	curl_close($curl);

        $xmldata = new SimpleXMLElement ($result);

        $source = $xmldata->ContentItem["source"];
	$power = true;
        switch ($source) {
            case "STANDBY":
                $mode = "Standby";
                $power = false;
            break;

            case "AUX":
                $mode = "AUX";
            break;

            case "STORED_MUSIC":
                $mode = "NAS";
            break;
			
            case "BLUETOOTH":
                $mode = "Bluetooth";
            break;

            case "INVALID_SOURCE":
                $mode = "Fehler";
            break;
			
            case "INTERNET_RADIO":
                $mode = "Internet Radio";
            break;
			
	    case "SPOTIFY":
                $mode = "Spotify";
            break;

            case "BLUETOOTH":
                $mode = "Bluetooth";
            break;
			
            default:
                $mode = "";
                $power = false;
	}

        // get now playing
        $nowplaying = utf8_decode($xmldata->stationName);

        // get description
        $description = utf8_decode($xmldata->description);

        // get logo
        $logourl = utf8_decode($xmldata->art);

        // return results       
        return array(
			"mode"    	=> $mode,
			"power"   	=> $power,
			"nowplaying"    => $nowplaying,
			"description"   => $description,
			"logourl"       => $logourl
		);
    }

// Volume
function get_volume()
    {
		global $DeviceIP;

        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL => "http://". $DeviceIP .":8090/volume",
            CURLOPT_HEADER => 0,
            CURLOPT_RETURNTRANSFER => 1));
        $result = curl_exec($curl);
        $xmldata = new SimpleXMLElement($result);
        $targetvolume = (integer)$xmldata->targetvolume;
        $actualvolume = (integer)$xmldata->actualvolume;
        $mutedvolume = (integer)$xmldata->muteenabled;
        // return results
        return array(   "targetvolume"  => $targetvolume,
                        "actualvolume"  => $actualvolume,
                        "mutedvolume"   => $mutedvolume);
    }

function set_volume($volume)
    {
		global $DeviceIP;

        $xmldata = "<volume>". $volume ."</volume>";
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL => "http://". $DeviceIP .":8090/volume",
            CURLOPT_HEADER => 0,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_POST => 1,
            CURLOPT_POSTFIELDS => $xmldata,
            CURLOPT_HTTPHEADER => array("Content-type: text/xml")));
        $result = curl_exec($curl);
        curl_close($curl);
    }

//////////////////////////////////////////

$Device = Status();

//////////////////////////////////////////

switch ($action)
{
	case "key":
		$value = strtoupper($value);
		PressKey($value);
	break;

	case "power":
		if (is_null($value))
		{
			PressKey("POWER");
			echo "Switch Power State";
		}
		else
		{
			if ($Device['power'] == 1 && $value == 0)
			{
				echo "Switch Power Off";
				PressKey("POWER");
			}

			if ($Device['power'] == 0 && $value == 1)
			{
				echo "Switch Power On";
				PressKey("POWER");
			}
		}
	break;
	
	case "getvol":
		$vol = get_volume();
		echo $vol['targetvolume'];
	break;

	case "setvol":
		set_volume($value);
		echo $value;
	break;

	case "volup":
		$vol = get_volume();
		$targetvol = $vol['targetvolume'] + $value;
		set_volume($targetvol);
		echo $targetvol;
	break;

	case "voldown":
		$vol = get_volume();
		$targetvol = $vol['targetvolume'] - $value;
		set_volume($targetvol);
		echo $targetvol;
	break;

	default:
	echo "Status: " . $Device[mode];
	echo "<br>";
	echo $Device[nowplaying];
	echo "<br>";
	echo $Device[description];
	echo "<br>";
	echo "<img src=" . $Device[logourl] . ">";
	break;
}

?>
