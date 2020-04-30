# Claening Library Data with OpenRefine
```
Author: Bree Norlander
Date: 2020-04-17
```
---
This will be a combination of exercises and learning about data retrieved from an Integrated Library System. We will start with circulation data that the Sno-Isle Library System in WA kindly provided to us. 

Sno-Isle uses [Polaris](https://www.iii.com/products/polaris-ils/) ILS. In order to gather six months of circulation by title data, the IT Manager ran the following SQL query to generate a csv:

```
select COUNT(distinct th.TransactionID) as [TotalCheckouts],
     br.BrowseTitle as [Title],
	   br.BrowseCallNo as [CallNumber],
	   br.BrowseAuthor as [Author],
	   o.Abbreviation as [OwningLibrary],
	   bs.data as [Publisher],
	   br.PublicationYear as [CopyrightDate],
	   MAX(ir.LastCircTransactionDate) as [DateLastBorrowed],
	   c.Name as [itype],
	   mt.Description as [ItemCode]
from PolarisTransactions.Polaris.TransactionHeaders th (nolock)
LEFT JOIN PolarisTransactions.Polaris.TransactionDetails tdmt (nolock)
	on th.TransactionID = tdmt.TransactionID and th.TransactionTypeID = 6001 and tdmt.TransactionSubTypeID = 4 --material type
LEFT JOIN PolarisTransactions.Polaris.TransactionDetails tdc (nolock)
	on th.TransactionID = tdc.TransactionID and tdc.TransactionSubTypeID = 61 -- collection
LEFT JOIN PolarisTransactions.Polaris.TransactionDetails tdi (nolock)
	on th.TransactionID = tdi.TransactionID and tdi.TransactionSubTypeID = 38 -- item record
LEFT JOIN Polaris.Polaris.ItemRecords ir (nolock)
	on ir.ItemRecordID = tdi.numValue
LEFT JOIN Polaris.Polaris.BibliographicRecords br (nolock)
	on br.BibliographicRecordID = ir.AssociatedBibRecordID
LEFT JOIN Polaris.Polaris.Organizations o (nolock)
	on o.OrganizationID = ir.AssignedBranchID
LEFT JOIN Polaris.Polaris.MaterialTypes mt (nolock)
	on mt.MaterialTypeID = tdmt.numValue
LEFT JOIN Polaris.Polaris.Collections c (nolock)
	on c.CollectionID = tdc.numValue
LEFT JOIN Polaris.Polaris.BibliographicTags bt (nolock)
	on bt.BibliographicRecordID = br.BibliographicRecordID and bt.TagNumber = 264
LEFT JOIN Polaris.Polaris.BibliographicSubfields bs (nolock)	
	on bs.BibliographicTagID = bt.BibliographicTagID and bs.Subfield = 'b'
where th.TranClientDate between DATEADD(MM, -6, GETDATE()) and GETDATE()
GROUP BY br.BrowseTitle, br.BrowseCallNo, br.BrowseAuthor, o.Abbreviation, bs.Data, br.PublicationYear, c.Name, mt.Description
```

Each ILS will have its own unique method for generating data, it won't always be a SQL query. This query resulted in 2,829,503 rows of data. It's a large dataset that is too big to store here in Github, so we've cut it down to a nice [sample](https://raw.githubusercontent.com/OpenDataLiteracy/LIS-598-Sp2020-DC2-testing/master/Data/SnoIsleCircSample.csv) of 100 rows to work with. If you click on the link, you will be able to see the data right in your browser which will be handy for reference, but for our next step, we will also be opening the data in the open-source application [OpenRefine](https://openrefine.org/). If you don't have it installed on your machine, you will find the download [here](https://openrefine.org/download.html).

OpenRefine is a tool designed to clean up messy data. Once you have it installed, open up the application (which will open and run in your default browser tab) and in the left menu choose "Create Project". OpenRefine has a handy tool for using data from a URL, so you can choose "Web Addresses (URLs)" to start your project and enter the link above to the data (https://raw.githubusercontent.com/OpenDataLiteracy/LIS-598-Sp2020-DC2-testing/master/Data/SnoIsleCircSample.csv). Click "Next."

![Screenshot of OpenRefine](Images/OpenRefineCreateProject.png)

OpenRefine automatically tries to detect the separator used in the file, but allows the use to change the detection paramaters.

![Screenshot of OpenRefine](Images/OpenRefineParseDataAs.png)

You'll notice that the file is in comma-separated value (CSV) format. Does that match the reality of the file? How can you tell?

If you look at the [raw data]((https://raw.githubusercontent.com/OpenDataLiteracy/LIS-598-Sp2020-DC2-testing/master/Data/SnoIsleCircSample.csv)) in your browser, and scan the first row, which in this case is the column header names, you will see that this file is NOT separated by commas, but rather by pipes (or [vertical bars](https://en.wikipedia.org/wiki/Vertical_bar)) |. This is not the most common of separator values, but OpenRefine still caught it and noted that the columns were separated by a custom value. If OpenRefine hadn't detected this, you would want to change this parameter before continuing on with the data. You should also note that OpenRefine detected the first row as the column headers and has chosen to fill empty cells with NULL. OpenRefine can also auto-detect data types. This may be useful when cleaning the data later, so go ahead and check the box next to "Parse cell text into numbers, dates, ..." These settings look fine for this dataset, but before continuing, take a look at the table shown above the parameters. Do you notice anything unexpected? How many columns has OpenRefine detected?

![Screenshot of OpenRefine](Images/OpenRefineColumns.png)

OpenRefine has detected 15 columns, 5 of which it has auto-named as "Column [#]". It appears that many of the cells in these columns are empty and not even filled with "NULL". When you scroll down a bit, you'll notice that row 23 is the first row to include data within columns 11-15. Continue scrolling and you'll notice that row 68 also contains data within those columns. Both of these entries are materials in Russian. Even if you don't read Russian, you should be able to notice that CopyrightDate and the DataLastBorrowed columns are not matching up with the data correctly in these two rows. It turns out the the Cyrillic alphabet contains a letter called "[Palochka](https://en.wikipedia.org/wiki/Palochka)" which is also the vertical bar. This is problematic for our file and will require "hand" fixing in a text editor (or Excel or Google Sheets, but I prefer a text editor). There may be workarounds to do this using only OpenRefine, but OpenRefine was not designed to manipulate individual rows and a text editor will be the fastest method.

There are many ways to import this file into a text editor. This tutorial will walk you through one way, but as described above you can also copy and paste from the browser data file or programmatically download the data file. Here we will continue using OpenRefine. Go ahead and click "Create Project" in OpenRefine. Next, "Export" the dataset as a "tab-separated value" file and save the file to your machine. Open the file in your favorite text editor (I like Visual Studio Code, but any text editor will work).

Your text editor probably won't recognize the first row as the column headers, so column headers are Row 1 (in OpenRefine Row 1 was the first row of data because it had recognized the Column Header row). This is important because we will have to add 1 to the row numbers we identified in OpenRefine as needing fixing. So, let's fix rows 24 (23+1) and 69 (68+1):

```
1	Meni͡	a zovut Sheĭlok : kaver-versii͡	a Venet͡	sianskogo kupt͡	sa Uilʹi͡	ama Shekspira	INTL-RUS FIC JACOBSO	Jacobson, Howard, author.	EDM	Izdatelʹstvo Ė,	2017	2020-01-03 11:21:07.667	International Russian	NULL
```
```
1	Zloĭ rok Mariny ?T	Svetaevoĭ : Zhiva?i	a dusha v mertvoĭ petle	INTL-RUS BIO TSVETAE POLIKOV	Polikovskai︠a︡, L. V. (Li︠u︡dmila Vladimirovna), author.	LYN	ĖKSMO,	2013	2019-09-26 16:18:23.823	International Russian	NULL			
```

We exported the file as tab-separated so all the pipes are gone and a tab indicates each next cell. For the two Russian titles that are causing problems, we need to add the pipes back in because they are legitimate characters within the titles. If you don't read Russian, how will you know where the title starts and ends? The first column is TotalCheckouts (an integer indicating how many times the title was checked out). So we know that the first integer is NOT part of the title. A tab then indicates we are entering a new cell. The second column should be the Title. We expect to see text or a "string" in this cell which is indeed the case. So how do we know where the title ends? The third column should be CallNumber, which we can see by looking at other examples in the dataset, is usually a combination of Dewey Decimal numbers and/or capital letters. In the case of the items above, we can identify what should be the third column by looking for text that is in all caps, both start with "INTL-RUS". Since we can now identify the start and end of the titles, we can replace the tabs that occur WITHIN the titles with the original pipe character. After fixing, the rows should now look like this:

```
1	Meni͡|a zovut Sheĭlok : kaver-versii͡|a Venet͡|sianskogo kupt͡|sa Uilʹi͡|ama Shekspira	INTL-RUS FIC JACOBSO	Jacobson, Howard, author.	EDM	Izdatelʹstvo Ė,	2017	2020-01-03 11:21:07.667	International Russian	NULL

```
```
1	Zloĭ rok Mariny ?T|Svetaevoĭ : Zhiva?i|a dusha v mertvoĭ petle	INTL-RUS BIO TSVETAE POLIKOV	Polikovskai︠a︡, L. V. (Li︠u︡dmila Vladimirovna), author.	LYN	ĖKSMO,	2013	2019-09-26 16:18:23.823	International Russian	NULL						
```

Now you need to save this file on your machine. Let's name it "SnoIsleCircSample.tsv". Now we can go back to OpenRefine and Create a New Project with this file. This time instead of importing the data from a URL, you are going to import the data from a file on your machine:

![Screenshot of OpenRefine](Images/OpenRefineGetDataFromFile.png)

Browse for your file that you named SnoIsleCircSample.tsv and click "Next." You will see a familiar screen. OpenRefine has auto-detected that the file is tab-separated this time and still recognizes that the first row contains column headings. Go ahead and choose "Parse cell text into numbers, dates, ..." as you did previously. Now take a look at the column headings. What do you notice? There are still extraneous columns! Why is this? When we exported the file as tab-separated, every cell was included (even the blank cells). When we manipulated the file in the text editor, we did not remove all the extra tabs that came at the end of each row. We could have manually gone through each row and deleted the extra tabs, but that would be tedious and OpenRefine offers and easier solution. BUt first let's double-check that our edits fixed rows 23 and 68. Scroll through the data and see if they end with the column "ItemCode." Indeed, it appears that columns 11-15 are now entirely empty - which is what we want. So for now, go ahead and click "Create Project."

Now we can begin cleaning the whole dataset with some simple options provided by OpenRefine. First, we know that Columns 11-15 are empty and we don't need them. If you click the down arrow next to the heading "Column11" you will get a menu of options. Choose "Edit Column" and then "Remove this Column". Repeat this for columns 12-15.

## Tidy Data

Now that we've cleaned the more egregious issues, let's take a look at this dataset through the lens of [tidy data principles](https://nniiicc.github.io/LIS-598-DC2-Sp2020/tidy-data.html). By fixing the Russian titles, we have cleaned the **values** in our dataset so that each value is in one cell and has a corresponding **observation** (row) and **variable** (column). Use the previous chapter on Tidy Data to assess the observations and variables. Is this a tidy dataset or do we need to perform any transformations?

I would argue that the dataset meets tidy data principles. But for the sake of example, let's practice separating a column. Let's take the column "DateLastBorrowed" and split it into two columns, "Date" and "Time" (this is not necessary for tidying data, it is common to include the full date-time within one column). To perform a column separation in OpenRefine, choose the down arrow next to the column you want to separate. 

![Screenshot of OpenRefine](Images/OpenRefineSplitColumn.png)

Next choose "Edit Columns" and "Split into Several Columns". We need to tell OpenRefine where to make the split in the data. For this dataset it is relatively simple: a space separates the date from the time. In the pop-up window, you will want to choose "How to Split Column" "by Separator" and the default separator is a comma. We want to replace that comma with a space. Then you can check "After splitting" "Guess cell type" and "Remove this column". This should have split the column in two, but autogenerated new column names, "DataLastBorrowed 1" and "DataLastBorrowed 2". Those column headers do not adequately describe the columns, so let's change them. For each of the two columns, click the drop-down menu and choose "Edit column" and "Rename this column". You can now rename them "Date" and "Time".

OK, I mentioned earlier that we didn't really need to split the DateLastBorrowed column. Let's return to the dataset pre-column splitting. OpenRefine includes a nice feature for capturing all the steps you've taken and allowing you to return to a previous version. In the left facet you should see a tab for "Undo/Redo". Click that tab and hopefully you see something like this:

![Screenshot of OpenRefine](Images/OpenRefineUndoRedo.png)

We want to return to the last step taken BEFORE splitting the column, which in this case is "Remove column Column 15". Click that step. You should now see DateLastBorrowed returned to its original format. You can now export this file as a tab-separated file for publishing to a repository.
