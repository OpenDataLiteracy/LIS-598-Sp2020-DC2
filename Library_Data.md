# Using Library Data
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

Each ILS will have its own unique method for generating data, it won't always be a SQL query. This query resulted in 2,829,503 rows of data. It's a large dataset that is too big to store here in Github, so we've cut it down to a nice [sample](https://raw.githubusercontent.com/OpenDataLiteracy/LIS-598-Sp2020-DC2-testing/master/Data/SnoIsleCircSample.csv) of 100 rows to work with. If you click on the link, you will be able to see the data right in your browser, you can copy paste the text into a text editor on your machine, or you can download the data to your personal machine (suggestions for how to download [here](https://stackoverflow.com/questions/4604663/download-single-files-from-github)). 
