<div align="center">

## load pcx


</div>

### Description

creates a GD image resource from a pcx file
 
### More Info
 
PCX version 5 files

$img = imagecreatefrompcx($filename);

GB image resource

returns false on error


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Ferry Timmers](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/ferry-timmers.md)
**Level**          |Intermediate
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |PHP 4\.0, PHP 5\.0
**Category**       |[Data Structures](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/data-structures__8-8.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/ferry-timmers-load-pcx__8-2564/archive/master.zip)

### API Declarations

Please inform me if you are going to use it. You may not include it in comercial products.


### Source Code

```
<?php
/******************************************
 * GD PCX reader             *
 *                    *
 * Author: Ferry Timmers         *
 *                    *
 * Date: 18-12-2008 20:22         *
 *                    *
 * Desc: Creates a GD image resource from *
 *    a specified pcx file.      *
 *    Needs PCX version 5       *
 *                    *
 ******************************************/
//------------------------------------------------------------------------------
function imagecreatefrompcx($filename)
{
	if (!$fp = @fopen($filename, 'rb'))
		return (false);
	// * * * Header * * *
	fread($fp, 1); // == 0x0A
	if (($version = ord(fread($fp, 1))) != 5)
	{
		fclose($fp);
		return (false);
	}
	$bbp = ord(fread($fp, 2));
	list($xmin, $ymin, $xmax, $ymax) = array_values(unpack('S4', fread($fp, 8)));
	$width = $xmax - $xmin + 1;
	$height = $ymax - $ymin + 1;
	//$size = $width * $height;
	fseek($fp, 54, SEEK_CUR);
	list($bpl) = array_values(unpack('S', fread($fp, 2)));
	// * * * Pallet * * *
	fseek($fp, -769, SEEK_END);
	if (ord(fread($fp, 1)) != 12)
	{
		fclose($fp);
		return (false);
	}
	$img = imagecreate($width, $height);
	for($i = 0; $i < 256; $i++)
	{
		list($r, $g, $b) = array_values(unpack('C3', fread($fp, 3)));
		$color[$i] = imagecolorallocate($img, $r, $g, $b);
	}
	// * * * Data * * *
	fseek($fp, 128, SEEK_SET);
	for ($y = 0; $y < $height; $y++)
	{
		$x = 0;
		while ($x < $bpl)
		{
			$c = ord(fread($fp, 1));
			if (($c & 0xC0) == 0xC0)
			{
				$c &= 0x3F;
				if (($c + $x) > $bpl)
					$c = $bpl - $x;
				$c += $x;
				$d = $color[ord(fread($fp, 1))];
				while ($x < $c)
				{
					imagesetpixel($img, $x, $y, $d);
					$x++;
				}
			}
			else
			{
				imagesetpixel($img, $x, $y, $color[$c]);
				$x++;
			}
		}
	}
	fclose($fp);
	return ($img);
}
//------------------------------------------------------------------------------
?>
```

