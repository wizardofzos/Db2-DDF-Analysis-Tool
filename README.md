# Db2-DDF-Analysis-Tool

Db2 DDF Analysis Tool is a set of DFSORT programs to report on Db2 Accounting Trace (SMF 101), specifically for DDF applications.

(When we say "DFSORT", the code has not been tested on any other sort product - but few if any problems are expected. Reports of compatibility would be gratefully received. In the rest of this guide we will use the term "SORT" - for both DFSORT and its competitors.)

**Notes**

1. The code relies on at least Accounting Trace Classes 1,2, and 3 on in the records.
There is further value in having Package-Level Accounting (Trace Classes 7,8, and 10) enabled - but this is not required.
2. You will need Db2 installed, or at least the Db2 installation macro library (commonly known as SDSNMACS).
This tool has only (recently) been tested with Db2 Version 12, though it has worked previously with Version 11.

## Repository Contents

This initial release consists of the following JCL programs:

* ASMEXIT - to assemble an E15 exit that reformats the SMF 101 records. (The $ASM procedure is included for this purpose.)
* BUILDDB - to read SMF 101 data and, using the exit and SORT to write a number of flat files. These files are what reporting code (generally SORT but could be eg REXX) can run against.
* SSIDCORR - to run a 2-step sample program that reports Db2 subsystem IDs and correlation IDs.

The intention is to add more reporting samples, and for users to generate their own. If they'd like to contribute them to this **open source** project that would be great.)

## Installation

To install the code:

1. Download from here.
1. Send the JCL members to a JCL PDS(E) on the z/OS LPAR you intend to run the code on.
1. Send the CTL member to another FB80 PDS(E) of your choosing.
1. Edit and submit the ASMEXIT job, assembling and link editing into a load library of your choosing.
`<SDSNMACS>` needs to be changed to point at your Db2 installation macro library.
1. Extract a small set of SMF 101 data and point at it (via SORTIN) in an edited version of the BUILDDB job.
1. Create a reporting data set, as outlined in [Reporting](#reporting).
1. Tailor and run the SSIDCORR sample reporting program.

All the jobs should return RC=0. The test with the small set of SMF 101 data should suffice for Installation Verification.

"Editing" means finding the variables, denoted by `<...>` and changing them to values that work for you.

Note the line

    DSN=<HLQ>.<QUAL2>.PMSERV.CTL(DDFIDSYM) 

This member is the SORT Symbols deck that the build job and reporting jobs will use to map the flat files created by BUILDDB.

## Use

In use there are two distinct phases:

1. Building the flat file database from raw SMF 101 records.
1. Reporting.

### Building The Database

You probably want to build the database more than just on a one-off basis.

Once you've established the BUILDDB job works you can modify it so the SORTIN points to an appropriate source.
Likewise you can modify the OUTFIL data sets to point to appropriate targets.

**Note:** If you have turned on Db2's Accounting Trace Compression you will need to decompress the records before passing them to the database build job.

### Reporting

Reporting jobs, obviously need to point to the right "database" input data sets.

Note again the need to use the edited name for `DSN=<HLQ>.<QUAL2>.PMSERV.CTL(DDFIDSYM)` to map the database data sets.

**Pro Tip:** You can concatenate your own symbols deck after this symbols file.
Generally I use inline symbols, but you can "harden" them in a file of your own.

You will need to allocate a report data set. It should be a PDS(E) with a LRECL of at least 4096 and a RECFM of VB.

Here's a sample JCL step to allocate the reporting data set.

    //ALDDFRPT EXEC PGM=IEFBR14
    //REPORT   DD DISP=(NEW,CATLG),
    //            SPACE=(CYL,(50,50,20)),UNIT=<UNIT>,
    //            DCB=(RECFM=VB,BLKSIZE=0,LRECL=4096),
    //            DSN=<HLQ>.<QUAL2>.DDF.REPORTS

If you follow the above naming convention sample reporting jobs should be able to allocate it, writing the reports to its members.

**Note:** Because this is RECFM=VB you could make the LRECL even larger than 4096.
There is no space wasted with a long LRECL because the RECFM is VB.