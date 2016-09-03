# [Cloudflare Email Protection Decode](http://pingturtle.com/home/post/cloudflare-email-protection-decoder0)

### All the credit goes to the original auther [Brad at pingturtle](http://pingturtle.com/home/post/cloudflare-email-protection-decoder).

Cloudflare (Link) is a pretty awesome way to speed up your site through their optimization and caching.  

They also keep a blacklist of abusive hosts and block them from accessing your site, be it botnet zombies, or spammers, etc etc.  

They have a feature where you can "scramble" all email addresses listed on your site to keep them hidden from email harvesting bots, which is pretty useful, especially for forums to give your users a little more protection when sharing info with each other, or having their emails listed in their profile (Tsk tsk.. forum software should scramble that stuff anyways, but a lot of them don't).

This got me curious as to what they did to hide the emails and decode them on the fly when the page is loaded. Its actually just a simple XOR encryption algorithm in Javascript with a random key set on every load. Below is their Javascript/span with encrypted email address as the class name...

``` 
<a id="__cf_email__" href="http://cloudflare.com/email-protection.html" class="f091809582839f9eb080999e97848582849c95de939f9d">[email protected]</a>
<script type="text/javascript">
/* <!--[CDATA[ */
(function(){try{var s,a,i,j,r,c,l=document.getElementById("__cf_email__");a=l.className;if(a){s='';r=parseInt(a.substr(0,2),16);for(j=2;a.length-j;j+=2){c=parseInt(a.substr(j,2),16)^r;s+=String.fromCharCode(c);}s=document.createTextNode(s);l.parentNode.replaceChild(s,l);}}catch(e){}})();
/* ]]--> */
</script>
```
... which is a lot easier to read when you format it ...

```
(function () {
    try {
        var s, a, i, j, r, c, l = document.getElementById("__cf_email__");
        a = l.className;
        if (a) {
            s = '';
            r = parseInt(a.substr(0, 2), 16);
            for (j = 2; a.length - j; j += 2) {
                c = parseInt(a.substr(j, 2), 16) ^ r;
                s += String.fromCharCode(c);
            }
            s = document.createTextNode(s);
            l.parentNode.replaceChild(s, l);
        }
    } catch (e) {}
})();
```
So anyways, I just took that function and re-wrote it in PHP and AutoIt. You need to feed the function the encrypted classname that is in the anchor tag that cloudflare inserts, and it will spit out the decoded email address. First, the php function...

``` 
<?php
echo 'Decoded Email: '.deCFEmail('f091809582839f9eb080999e97848582849c95de939f9d');
 
function deCFEmail($c){
   $k = hexdec(substr($c,0,2));
   for($i=2,$m='';$i<strlen($c)-1;$i+=2)$m.=chr(hexdec(substr($c,$i,2))^$k);
   return $m;
}
```

And the AutoIt version...

``` 
Dim $enc = "f091809582839f9eb080999e97848582849c95de939f9d"
 
MsgBox(0,"","Decoded Email: " & deCFEmail($enc) )
 
Func deCFEmail($c)
    $m=''
    $k = Dec(StringLeft($c,2))
    For $i = 2 To StringLen($enc)-2 Step 2
        $m&=Chr(BitXOR(Dec(StringMid($c, $i+1, 2)),$k))
    Next
    Return $m
EndFunc
```
... and that's it! I suppose if you wanted to extract emails, you could write a regular expression to match the classname and the anchor tag that cloudflare uses, and extract/decode emails on the fly.

