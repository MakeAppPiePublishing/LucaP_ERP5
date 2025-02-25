# LucaP_ERP5
THe next step in the Luca P ERP story
# LucaERP part 5: Building A form
#LucaP
In this series of biweekly columns, we're building an ERP from scratch, starting with the general ledger. We've learned about the Part of UI, and now it's time to make a UI for a form. We'll build the Chart of Accounts entry form for our first form. 

I'll assume you've read the last one or know all the UI components and types I've discussed before. For even more on types, check out the first part of my [HANA functions newsletter](https://www.linkedin.com/pulse/bizoneness-common-sql-hana-types-operators-functions-steven-lipton-ipdpc). I will post the code to [GitHub](https://github.com/MakeAppPiePublishing/LucaP_ERP5) but not show it here. I'll post screenshots of what I'm discussing. 

# part 1: Field Components

We'll have to define our field components first. I want something that looks like this: 

It has a title and an input area. HTML's **\<INPUT\>** will only give the Text field, not the label. I get the label inside the field like this: 

I want a consistent look for everything; thus, I'll have some formatting to do in the iPad environment. 

One wrinkle in all this I didn't mention was inactive fields. An inactive field does not accept input.   You may have a form that, for security purposes, is read-only. Some ERP systems have read-only modes, while others have *trackers*, which are separate but read-only modules. 

You might also have read-only fields. Your primary key after entry will be one example where we do not want this changed. Some items on a closed document will become read-only as well. For example, modifying the credits and debits in the chart of account types is a bad accounting practice.

I'll make my inactive fields  with grayed-out fields like this: 


I separate the code into the functional code and the style modifier. Setting standardized styles makes your work easier. Having one place to make changes in appearance limits the work of changing down every formatting issue you do in an application.  

I do the same thing for other fields. I build input fields for integers, floating point numbers, and currency similarly so they look the same as my text fields. 

I'll use a checkbox for yes/no answers. That doesn't fit my formatting, so I must code that one myself. 
   
It turns on and off a graphic of a box and a square. 
   
Three's several pickers. I'll also add a string picker, a picker using a number, title, and date. All work the same as the text fields. 
   
The last field I added is a text field with a pop-out selector control. If I want to find a specific item on a longer list, I'll use this one to give me browsing capabilities. 
   
# Layout
   
Form layout can take a few designs. A common one is this one, with a side menu bar. 
   
This layout is especially good for systems that use multiple open windows. LucaERP will not have only one window, so we'll use a different setup with module navigation at the top. 
   
Both have a two-column layout for the fields and a toolbar above the fields for field navigation and other functions we'll cover later. The information bar and the action buttons for accept and cancel are at the bottom. 
   
In swiftUI, I made a general template like this in the code: 

```
VStack {
            titleView
            navView
            toolbarView
            HStack{
                bodyView
                bodyView
            }
            infoView
            Spacer()
        }
```

   
For our template, we already have the title code; we've been using this: 
   
I'll skip the navigation and toolbar and go to the body. We use the fields we defined to do this. I define a series of variables for the fields we have. 

Some might think we use the columns directly for Accounts here, but that would cause some problems. Most notably, we lose control of updating and canceling updates. For many database and persistence systems, changing a value in a field changes it in the database. We want to control when we press a button to add, modify, or cancel. So, I made local variables to hold the values displayed and will change and update these values when necessary. 


Once I add the variables, I can add the fields using them. In our case, we'll use it there. 
   
Do notice the text field with the magnifying glass. This is for a combination text field and picker  I used on the Account Number. It will have an extra button to open another window for a larger selection than a standard picker. To display more than a picker can, I revised the ChartOfAccountsView we made earlier to show only the account number and name. 

That builds our body. The toolbar is the next step, where we place several critical buttons. The first set is the field navigation buttons, which will move us between accounts. If working from a database, many of the actions in the tools are taken care of for you -- you'll add the action to a button. Unfortunately, we have to write this logic for programming languages like Swift. 

To do this in the most compatible way possible, I did a few things: 
1. Sort the data by id. Count how many records there are. That makes a range between zero and this count. 
2. Use an index. If the index does not exist, this is a new record. If the index is invalid, jump to the last record. 
3. Move the index to the beginning, previous, next, or last index depending on the button pressed. 
4. When the index changes, the data in the fields change accordingly. 

This plan allows the traverse of any list without messing with its type. It uses only an index pointer to the location. 

Incrementing and decrementing the index works, but using it runs into problems at the ends of the sequence. If you have the sequence
```
0 1 2 3 4 5
```
and you are at 5, then add one, where do you go next? Similarly, where do you go next if you are at 0 and subtract 1? 

Here, we'll either stop at 0  and 5  or loop back to the other end, so after 0 comes 5, and after 5 comes 0. I did the looping approach. The *modulo*  operator **%**, also known as the *remainder*, can make this easy. **7 % 5  = 2** for example. If do 

```
index % count
```
I will always stay within the range of numbers. 

That leads to another issue: How do I know when an entry is new? While I tried many different approaches to writing this, the best way is to use an optional value. Optionals have a special value, often referred to as **nil** or **Null**, which means there is no value. An index of **nil** is a new record. I'll add one more button to my four navigation ones to set the index to nil when the user adds a new record. 

The last navigation issue is when a record is not in the range, and we access it directly. I added a filter to turn these into the last record. 

I'll also add a button to toggle between edit and read-only modes. I'll ensure the user gets out of read-only when pushing the add record button. 

With the buttons set up, I can add the toolbar. Once the bar is there, I can use the index to move between records. 

Here we are, back to the updating problem we had earlier. When the index changes, we retrieve the account record at that position and populate the data into the fields. If the index becomes nil, we blank the record. When any of the navbar buttons we've set changes, we display a new data set. 

Finally, I'll add the info bar. I'll add status messages here and the two action buttons. The first is the add/change/okay button, which changes names and functions depending on context. If in new record mode, the button displays **Add*. In read-only mode, the button will display **Okay**. Finally, the button will read **Change**if in edit mode. 

With all of that, we have the following layout : 


This is the basis of a form. We did a simple one this time, and there are still a few functions to set up. In two weeks, we'll get those bottom buttons working and explain why you want CRUD in this form. We'll have our basic template for all ERP forms when we do that. Next week, I'll finish my HANA survey with predicates.  
 




 
 

