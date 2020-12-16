# Gootkit

Gootkit is a fileless malware that has been heavily targeting Germany in 2020. Gootkit stores its DLLs inside the Windows Registry from which they can be extracted.

## Recipe input

This recipe takes as input a Registry Key from an infected system as exported by, e.g., ```regedit```. In the following example that would be the file ```acfbbcbefecadce.reg``` which looks something like:

```
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\SOFTWARE\acfbbcbefecadce]
@=""
"0"="$Command =[System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String(\"JABF[...]
"1"="XAHoAUgBRAE8ANwBNAHYAUABDADAAbQBwAHIAVQArAEQANAA1A[...]
"2"="VAE4AUgBCADkAWABYADAAZAA4AHkALwBYADAAOQAvAGIAMwBJA[...]
"3"="wADIARQA2AFAAZwB6AEUAMABrADYALwBxADIASgBIAEIAMwAvA[...]
[...]
"168"="vAGsAWgArAC8ASQBPAC8AUwBpAFAAUABuAFQAUAArAFoAaAB[...]
"169"="rAFMAUABFADAAdAAvAHYAYQBXADQAbQBlAHEAagBYAG0ARwB[...]
"171"="vAC8AMAB2AGEAYQBxAEIAOQ[...]A0ACgA=\")); Invoke-Expression $Command;Start-Sleep -s 22222;"
```

## Stage 1

Use CyberChef's *Open file as input* button to load the file ```acfbbcbefecadce.reg``` (you may want to disable *Auto Bake* beforehand).

Copy the following recipe to *Load Recipe* and press *BAKE!*

```
[{"op":"Decode text","args":["UTF-16LE (1200)"]},{"op":"Regular expression","args":["User defined","[a-zA-Z0-9+/=]{30,}",true,true,false,false,false,false,"List matches"]},{"op":"From Base64","args":["A-Za-z0-9+/=",true]},{"op":"Decode text","args":["UTF-16LE (1200)"]},{"op":"Regular expression","args":["User defined","[a-zA-Z0-9+/=]{30,}",true,true,false,false,false,false,"List matches"]},{"op":"From Base64","args":["A-Za-z0-9+/=",true]},{"op":"Raw Inflate","args":[0,0,"Adaptive",false,false]}]
```

The *Output* panel will show the extracted DLL (which contains another obfuscated DLL and which will act as input for Stage 2):
- https://www.virustotal.com/gui/file/973d0318f9d9aec575db054ac9a99d96ff34121473165b10dfba60552a8beed4/detection
- https://www.hybrid-analysis.com/sample/973d0318f9d9aec575db054ac9a99d96ff34121473165b10dfba60552a8beed4/5f8731c48ad8847f167829b6
- https://app.any.run/tasks/30f760b9-f6a6-44d6-b2fb-a7325f6b05ca/


## Stage 2

Use the DLL as input for Stage 2 (Note: if CyberChef's *Replace input with output* doesn't work simply *Save output to file* at the end of the previous step, then use *Open file as input* again).

Use another recipe:

```
[{"op":"Strings","args":["16-bit littleendian",4,"All printable chars (A)",false]},{"op":"Decode text","args":["UTF-16LE (1200)"]},{"op":"Regular expression","args":["User defined","[a-zA-Z0-9+/=]{30,}",true,true,false,false,false,false,"List matches"]},{"op":"From Base64","args":["A-Za-z0-9+/=",true]}]
```

Again, baking will lead to a DLL:
- https://www.virustotal.com/gui/file/60aef1b657e6c701f88fc1af6f56f93727a8f4af2d1001ddfa23e016258e333f/detection
- https://app.any.run/tasks/5cb84067-ea53-4eee-8093-cbccb93d011a/

## References
- https://www.trendmicro.com/en_us/research/20/l/investigating-the-gootkit-loader.html
- https://blog.malwarebytes.com/threat-analysis/2020/11/german-users-targeted-with-gootkit-banker-or-revil-ransomware/
- https://malpedia.caad.fkie.fraunhofer.de/details/win.gootkit

## Kudos
Thanks a lot to [mattnotmax](https://github.com/mattnotmax/) for the inspiration!
