Structs From Tables ReadMe
==========================

VERSION 1.0
===========

REQUIRES
========

VDF 11.0 or higher.  In fact I have only tested it with VDF 11.0, but it should
work perfectly well with 11.1 AFAICS.  It also seems to work with the current
VDF 12 beta.


DESCRIPTION
===========

StructsFromTables is a Visual DataFlex (tm) program which generates 
structs (available since VDF 11.0) from your database tables, and also
global methods for working with them.

The interface consists of a Windows (tm) dialog, containing a form, a
grid and a number of buttons.

The form allows you to select the file that the structs and methods will
be generated in.  By default this is .\DDSrc\TableStructs.pkg in your
current workspace, but you can change it whatever you want.

The grid will contain a list of tables from the current workspace's
FileList.cfg, and, if the file you are working with already exists, the
currently generated entries will be already checked for you, so that you
don't inadvertently lose any you are currently using.

You check the tables you want to generate structs and methods for, then
click "Generate", saying "Yes" to the "Continue?" prompt, and "OK" to the
"Done" notification.

The structs and methods (global procedures and functions) for the tables
you check will then be generated in the file shown in the form at the
top of the dialog - you can open it in the studio and look at them, but
DON'T edit the generated file manually!  (Well, you can do what you like
of course, but don't then complain you were not warned!)

For each table you chose there will be:

   1. A STRUCT having the same layout as the table: 't<TableName>'.

   2. A FUNCTION to move data from the file buffer to a struct variable and
      return that variable: '<TableName>_BufferToStruct'.

   3. A PROCEDURE which takes a struct variable and moves its data to the
      file buffer: '<TableName>_StructToBuffer'.

   4. A FUNCTION which loads a struct variable from the Current_Field_Value
      properties of a passed Data Dictionary handle and returns it:
      '<TableName>_DDToStruct'.

   5. A PROCEDURE which takes a struct variable and a Data Dictionary handle
      and sets the DD Field_Changed_Value properties from the struct data:
       '<TableName>_StructToDD'.


These allow you to move data to and from DataFlex file buffers (2 & 3) and
struct variables of the associated type (1), or to and from Data Dictionary
properties (4 & 5) and the struct variables.

Usage Example:
--------------

(using the VDF 11 Order Entry Sample workspace)

//==============================================

Open Customer

Use TableStructs.pkg

Procedure ShowCust Global Integer iNum tCustomer ThisOne
   ShowLn "=================================="
   ShowLn ("ShowCust, record" * String(iNum))
   ShowLn "Number:      " (String(ThisOne.Customer_Number))
   ShowLn "Name:        " ThisOne.Name
   ShowLn "Address:     " ThisOne.Address
   ShowLn "City:        " ThisOne.City
   ShowLn "State:       " ThisOne.State
   ShowLn "=================================="
   ShowLn
End_Procedure  // ShowCust

Procedure DoStuff Global Integer iNum
   tCustomer Cust

   Get Customer_BufferToStruct to Cust

   Send ShowCust iNum Cust
End_Procedure  // DoStuff

Integer giRec
String  gsAKey

Clear Customer

For giRec From 1 to 5
   Find gt Customer by Index.2
   If (Found) Send DoStuff giRec
   Else Break
Loop

InKey gsAKey

Abort

//==============================================

Here the outer code finds Customers in a loop, fires DoStuff, which calls the
Customer_BufferToStruct function in TableStructs.pkg to fill a variable of type
tCustomer (defined in TableStructs.pkg) from the file buffer, then passes that
variable on to ShowCust to display.


NOTES
=====

1. The program will detect a change in current workspace made while it is running
and will automatically switch to the new current workspace's filelist.

2. I put it on the "Utilities" pull-down in my VDF studio, so I can run it
whenever the situation calls for it.  It is after all a development utility.


WHY WOULD YOU WANT TO DO THAT?
==============================

There are at least two reasons for using this kind of approach (I know there
are at least two, because it wasn't until I encountered the second one that I
was prompted to write the program) - other people may come up with others.

-----

The first is a web services scenario where you want to pass complete records, or
even sets of records (which would be arrays of structs), between applications.

One example of this might be the synchronization of data between multiple sites,
using a VDF Web Service at the "remote" end and a web service client at the
local end.

The generated structs would be used to define the web service, while that
definition would then be exposed as WSDL allowing the web service client to map
to it.  The web service client could then use the methods in TableStructs to
load its buffer or data dictionary data into the struct to be passed.

[NOTE: there is an issue here, because of the way VDF generates its web service
client class packages: it renames the structure it detects in a service's WSDL
by prefixing it with "tWS".  The generated web service client package would 
therefore need to be modified after generation to (a) Use TableStructs.pkg,
(b) change the names of the structs used (Find and Replace "tWSt" with "t" will
do in simplest case, where the table-structs are the only ones involved), and
(c) comment out the structs defined in the package itself (or at least the ones
made redundant by TableStructs.pkg).  Once you know what you are doing this boils
down to a couple of minutes work for each "generation" of a web service client.]

As records are modified on the local system, the web service client can request
that the record be changed (or inserted, or deleted, but you don't need a struct
for the last - just a unique key) by the service, passing it a struct containing
the modified record.

-----

The second is a situation where you want to encapsulate logic working on data,
from the usual finding and updating logic, probably for reasons of reuse, but
also for clarity.

An example would be components for reading data into a system from multiple
different external sources.  The reading and mapping logic would be different
for each source, but the updating logic would be the same for all.  The data
mapping components would send their data to the update logic via structs with
the same layout as the database tables.  The update logic would be completely
ignorant of the reading and mapping logic; the reading logic would only have to
worry about what it was reading; only the mapping logic would need to know both
sides of the story, but even it wouldn't need to know the internals of the
update component.

-----

In both cases, it wouldn't be terribly difficult to manually write the structs
and methods, but if modifications were then made to the involved tables, the
maintenance overhead - and the concomitant probability of introducing errors -
would be considerable.  With StructsFromTables you can just regenerate your
TableStructs.pkg file after making any database changes, significantly reducing
the maintenance overhead.


SOURCE
======

The source consists of two source files:

	StructsFromTables.src - the program source
	StructFromTable.dg    - the on-screen bit: does most of the work

You should be able to put the two source files into the AppSrc directory of a
project (or create a new project for them) and just run the compiler on the .src
to produce an executable.

You can register the files with the VDF Studio (Component -> Register Component)
as an Application and a Dialog respectively.  Having done that you should be
able to open them in the studio, however the dialog will give an error reading
the source code, because the studio doesn't understand the BasicPanel class, and
will open in full source mode.  To be able to manipulate the dialog in component
mode, you should just (manually) change the class of the oStructFromTab object
from BasicPanel to ModalPanel, then F11 to switch to component outline mode.
Remember to change this back, then save and recompile, when you have finished
making your changes.


LIMITATIONS
===========

The program will not currently handle BINARY or OVERLAP fields.  In most cases
the lack of overlaps will not be an issue, as the underlying fields will be
handled.  I'm not sure what to do about BINARY fields.


KNOWN BUGS
==========

None at the moment, but if people start to use it I feel sure I'll be hearing
about them.  Version 1.1 may be just around the corner!  ;-)

Mail them to mpeat@unicorninterglobal.com.


CONTACT
=======

If you have any problems or suggestions, I can be contacted at:

	mpeat@unicorninterglobal.com


COPYRIGHT
=========

This Software is distributed under the open source MIT License, as set out 
below:

Copyright (c) 2006 Mike Peat

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in 
the Software without restriction, including without limitation the rights to 
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies 
of the Software, and to permit persons to whom the Software is furnished to do 
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all 
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE 
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, 
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE 
SOFTWARE.
