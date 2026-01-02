# quinary.exe

## What is `quinary.exe`?

The file `quinary.exe` is a valid x86-64 Windows executable.
```
$ file quinary.exe
quinary.exe: quinary.exe: PE32+ executable for MS Windows 5.02 (console), x86-64 (stripped to external PDB), 11 sections
```

The file `quinary.exe` is also its own source code, written in C.
```
$ gcc -fpermissive -w -x c quinary.exe
```

Finally, `quinary.exe` is a BASH script that contains all the steps required to build itself.

```
$ bash quinary.exe
Building a.exe...
Success!
```

This makes the program a [quine](https://en.wikipedia.org/wiki/Quine_(computing)) (of sorts) as well as a [polyglot](https://en.wikipedia.org/wiki/Polyglot_(computing)).

## What does `quinary.exe` do?

When used as an executable, quinary runs itself as a BASH script, which compiles itself as C source, producing an identical executable as output.

```
$ ./quinary
sh C:\Users\mshamas\dev\projects\quinary\quinary.exe
Building a.exe...
Success!

$ diff quinary.exe a.exe

```

To specify the output filename, use `./quinary output.exe` (or `bash quinary.exe output.exe`).

## Requirements

- TBD

## How it Works

The following shows the strings significant to the source code and script.

```C
MZ/* '*/()/*' 2>/dev/null <<"EF" #*/{}
static void*a=R"p(

[remainder of MS-DOS header, MS-DOS stub, and PE headers appear here]

)p";
#include <stdlib.h>
main(int c,char**v){char b[99]={};sprintf(b,"sh %s %s",v[0],c>1?v[1]:"");puts(b);system(b);}static void*d=R"p(
EF
P=${1:-a.exe}
set -e
echo "Building $P..."
gcc -Os -s -w -fpermissive -Wl,--format=pe-x86-64,--file-alignment=2048,--no-insert-timestamp,-gc-sections -x c $0 -o $P
D="dd conv=notrunc if=$0 of=$P bs=1 status=none"
$D count=60
$D skip=1536 seek=1536 count=512
echo ")p\";" >> $P
echo "Success!"
exit

[PE sections appear here]

)p";
```

The first string replaces part of the MS-DOS header at the beginning of the file. Luckily, the only essential fields read by Windows are the 'MZ' signature and the e_lfanew address (pointing to the PE headers). Both of these fields are preserved, and otherwise, the string only serves to escape the binary data that follows.

The second string is inserted into padding data following the PE headers. When interpreted as BASH, this will use gcc to compile itself. The compiler/linker flags minimize the binary size, silence warnings, and ensure the output binary is formatted correctly. When interpreted as C source, this creates a small executable that runs itself as a bash script.

The file contains many non-printable characters that violate ISO C standards and BASH syntax rules. To circumvent this, GCC's raw string literal extension is used to capture any binary data. In BASH, a heredoc is used similarly.
