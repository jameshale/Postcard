#### Create a data grid#### 

Each data grid consists of a group with lots of scripts and objects.
Luckily, you do not need to recreate all that from scratch. Instead you
can simply copy a template data grid from the data grid library. Because
this library is neccesary for all data grid functionality, it will be
added to standalones by the IDE automatically. Please also see 
[What Do I Need to Do To Deploy a Standalone With A Data Grid?](#What-Do-I-Need-to-Do-To-Deploy-a-Standalone-With-A-Data-Grid?) 
for an explanation when and how you need to add the library to
standalones yourself.

The code to copy a datagrid:

    --copy the template group
    copy group "DataGrid" of group "Templates" of stack "revDataGridLibrary" to this card
    set the name of it to "my Datagrid"
    set the dgProp["style"] of group "my Datagrid" to "table"

#### Create the Row template#### 

In theory, your data grid is already ready for use. But for the IDE to
interact correctly with the new data grid, you also need to add a
substack with the row template, just as the IDE would. On the second
card of that stack, you will need to add a group called **Row Template**,
and you need to point your data grid to that group.

Here is how you could do that:

    --create row template
    put the name of this stack into theMainstack
    put "Data Grid Templates" && the seconds into theName
    create invisible stack theName
    set the mainStack of stack theName to theMainstack
    go stack theName
    create card
    create field "label"
    create graphic "Background"
    group field 1 and graphic 1
    set the name of last group to "Row Template"
    go stack theMainstack
    --point datagrid to template
    set the dgProp["Row Template"] of group "my Datagrid" to the \
     long id of group "Row Template" of stack theName

#### Fill in Your Data ####

As a last step, you will want to insert the prepared data into the data
grid. Again, this assumes that the first row of your data contains an
unique description for each column:

    --add data
    put true into firstLineIsNames
    set the dgText[firstLineIsNames] of group "my DataGrid" to theData
   

#### Prepare the Columns of the Data Grid ####

Because you are creating the datagrid from arbitrary data, you will also
need to ***dynamically create*** all necessary columns. To do that you
need to set the **dgProp["columns"]**. Note that it accepts the names only
as a return delimited list.

If your data includes the column names in it's first row, you can simply
take this and replace tab with return:

    --create columns
    put line 1 of theData into theColumns
    replace tab with return in theColumns
    set the dgProp["columns"] of group "my DataGrid" to theColumns

#### Everything together#### 

To test out the example scripts, simply copy the following **mouseUp**
handler into a button of an empty stack, and click on it with the
**browse** tool.

    on mouseUp
      answer file "tab delimited" with type "any file" or \
       type "tab file|tab|TEXT" or type "txt file|txt|TEXT"
      --in case the user cancels the file dialogue, the script does not proceed
      if it = "" then
         exit mouseUp
      end if
      put url ("file:" & it) into theData
    
      lock screen --when doing visual stuff, this helps to speed things up
      --make sure the group does not exist already
      if there is a group "my Datagrid" then
           delete group "my Datagrid"
      end if
      --copy the template group
      copy group "DataGrid" of group "Templates" of stack "revDataGridLibrary" \
       to this card
      set the name of it to "my Datagrid"
      set the dgProp["style"] of group "my Datagrid" to "table"
      --position it nicely
      set the rectangle of group "my Datagrid" to 0,the bottom \
       of me + 16,the width of this card, the height of this card
      
      --create row template
      put the name of this stack into theMainstack
      put "Data Grid Templates" && the seconds into theName
      create invisible stack theName
      set the mainStack of stack theName to theMainstack
      go stack theName
      create card
      create field "label"
      create graphic "Background"
      group field 1 and graphic 1
      set the name of last group to "Row Template"
      go stack theMainstack
      --point datagrid to template
      set the dgProp["Row Template"] of group "my Datagrid" to the \
       long id of group "Row Template" of stack theName
    
      --create columns
      put line 1 of theData into theColumns
      --no empty column names are allowed, so insert a space when \
       that happens
      replace tab & tab with tab & " " & tab in theColumns
      replace tab with return in theColumns
      set the dgProp["columns"] of group "my DataGrid" to theColumns
    
      --add data
      put true into firstLineAreNames
      set the dgText[firstLineAreNames] of group "my DataGrid" to theData
    end mouseUp

## Data Grid Tips & Tricks ##

### How Do I Scroll a Row to the Top of the Data Grid Form? ###

This lesson will show you how to scroll a particular row to the top of the Data Grid form 
using the **dgRectOfIndex** (or **dgRectOfLine**) property. This technique is useful when your 
rows are not a fixed height.


Here is selected row in a data grid (1). 
The goal is to scroll that row to the top of the data grid (2).

####How To Do It####

Here is the code that will scroll the selected line to the top of the data grid.

***Copy & Paste***

    put the dgHilitedIndex of group "DataGrid" into theIndex
    put the dgRectOfIndex [theIndex] of group "DataGrid" into theControlRect
    put the rect of group "DataGrid" into theGridRect
    put item 2 of theGridRect - item 2 of theControlRect into theOffset
    set the dgVScroll of group "DataGrid" to the dgVScroll of group "DataGrid" - theOffset

#### The Result ####

[link to ward top](#Create-the-Row-template)

After executing the code the row will have been moved to the top of the data grid.

### How Do I Find Out Which Line the Mouse Is Over? ###

Each data grid row has a **dgDataControl** property that returns the **long id** of the 
control. Each control also has a **dgLine** and **dgIndex** property. You can combine 
the two in order to identify the control the mouse is over.

    put the dgDataControl of the mousecontrol into theControl
    if theControl is not empty then
      put the dgLine of theControl into theLineNo 
    end if
   

### Converting a Database Cursor to a Data Grid Array ###

This lesson demonstrates a handler that will convert a database cursor into an array that 
you can use to set the **dgData** property of a data grid.
