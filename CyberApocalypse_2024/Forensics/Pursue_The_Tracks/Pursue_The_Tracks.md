# Persue The Tracks

## Description
> Luxx, leader of The Phreaks, immerses himself in the depths of his computer, tirelessly pursuing the secrets of a file he obtained accessing an opposing faction member workstation. With unwavering determination, he scours through data, putting together fragments of information trying to take some advantage on other factions. To get the flag, you need to answer the questions from the docker instance.

<br>
<br>

## Walkthrough

In this challenge, we were given a questionnaire about a Netcat session, as well as a __Master File Table__ (MFT) file.

An MFT file is an important component in the Windows NTFS file system. It acts as a central data structure that stores the metadata of all files and directories on an NTFS-formatted partition.

Important information can be obtained from an MFT file, such as timestamps, i.e. when files were created and/or modified.

It also contains information on whether a file has been "deleted" on the computer, where the file is physically located on the data storage device and its size.

Furthermore, files can even store themselves in an MFT if they are very small (resident files).

<br>

### Solution

I used two tools to analyze the file:

- "MFTExplorer" (Windows)
- "mft.pl"  (Linux)

<br>

### Questionnaire & Flag

```
+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+
|       Title       |                                                                    Description                                                                    |
+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+
| Pursue The Tracks |                                    Luxx, leader of The Phreaks, immerses himself in the depths of his computer,                                   |
|                   |                      tirelessly pursuing the secrets of a file he obtained accessing an opposing faction member workstation.                      |
|                   | With unwavering determination, he scours through data, putting together fragments of information trying to take some advantage on other factions. |
|                   |                                    To get the flag, you need to answer the questions from the docker instance.                                    |
+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+

Files are related to two years, which are those? (for example: 1993,1995)
> 2023,2024
[+] Correct!

There are some documents, which is the name of the first file written? (for example: randomname.pdf)
> Final_Annual_Report.xlsx
[+] Correct!

Which file was deleted? (for example: randomname.pdf)
> Marketing_Plan.xlsx
[+] Correct!

How many of them have been set in Hidden mode? (for example: 43)
> 1
[+] Correct!

Which is the filename of the important TXT file that was created? (for example: randomname.txt)
> credentials.txt
[+] Correct!

A file was also copied, which is the new filename? (for example: randomname.pdf)
> Financial_Statement_draft.xlsx
[+] Correct!

Which file was modified after creation? (for example: randomname.pdf)
> Project_Proposal.pdf
[+] Correct!

What is the name of the file located at record number 45? (for example: randomname.pdf)
> Annual_Report.xlsx
[+] Correct!

What is the size of the file located at record number 40? (for example: 1337)
> 57344
[+] Correct!

[+] Here is the flag: HTB{p4rs1ng_mft_1s_v3ry_1mp0rt4nt_s0m3t1m3s}
```
