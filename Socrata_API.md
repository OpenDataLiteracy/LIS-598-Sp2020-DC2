# Using the Socrata API to Access WA State Data
```
Author: Bree Norlander
Date: 2020-04-03
```
---
## Data.wa.gov

In this demonstration and exercise, we'll look at the [State of Washington's data portal](https://data.wa.gov/), which is built on a [Socrata](https://www.tylertech.com/products/socrata/data-platform) platform. Socrata has created a custom API called SODA (Socrata Open Data API) which is very similar to a REST API. They also have a custom query language called SoQL or “Socrata Query Language”. It will be important to refer to the [SODA API Documentation](https://dev.socrata.com/consumers/getting-started.html) throughout this demonstration.

Start by visiting the state data portal homepage at [https://data.wa.gov/](https://data.wa.gov/). There are actually several different portals listed here on the homepage. For this demo we are going to look specifically at the Socrata platform which can be accessed by clicking on "Data Catalog."

![Screenshot showing Data Catalog Link](Images/Data_Catalog_Screen.png)

Here you will see everything available on the data catalog, from datasets, to maps, to filtered views, and more. We will specifically be looking at Datasets. So go ahead and filter your view to just Datasets by clicking "Datasets" in the View Types of the faceted search interface on the left of the screen. Here you see all of the datasets hosted on the state data portal. Let's take a look at a specific example. Search for "Cleanup Sites in Washington State" and filter again to Datasets. We're looking for the dataset with that specific title, which you should be able to find [here](https://data.wa.gov/Natural-Resources-Environment/Cleanup-Sites-in-Washington-State/vtkh-65is).

![Screenshot Cleanup Sites in Washington State Dataset homepage](Images/Cleanup_Sites_Screen.png)

Take some time to familiarize yourself with what is available on this page including metadata, data preview, and content made with this data. When you have an idea of the data that's available in the dataset, click the API link near the top of the page.

![Screenshot API Link](Images/API_Link_Window.png)

If you click "Copy" on the API endpoint and paste this URL into a browser window, you will see all the data. Depending on which brower you use, you may see this JSON file formatted in an easy to navigate format or a raw textual format. This is nice, but we could have just exported the data and viewed it in our preferred software. The API endpoint is an easy way to import data programatically into data analysis tools such as Tableau and R. But you can also use to API to filter the data to exactly what you need, to avoid downloading or importing files that are too large. So we have the api endpoint URL [`https://data.wa.gov/resource/vtkh-65is.json`](https://data.wa.gov/resource/vtkh-65is.json), let's figure out how to filter the data to see only sites which are in King County.

If you look at the data portal page again, you will see that there is a list of all the columns in the dataset. One of the columns is "County Name." Click on the arrow to the right of "County Name":

![Screenshot County Name](Images/County_Name_Screen.png)

You can now see that the data type for this colummn is "Text" and the API Field Name is "county_name". This API field name is what you'll need for doing a filtered API call. To figure out the structure of our filtered API call, let's refer to the SODA API Documentation, specifically the [Getting Started](https://dev.socrata.com/consumers/getting-started.html) chapter. If you scroll down (or better yet read through) to the "Building simple filters and queries" section, the first example for a simple filters will give us the information we need.

![Screenshot Simple Filters](Images/Simple_Filters_Screen.png)

You can see by this example, that the beginning of the query (the api endpoint) is the same as what we used above: [`https://data.wa.gov/resource/vtkh-65is.json`](https://data.wa.gov/resource/vtkh-65is.json). The next step to adding a query to the URL is to add a question mark. This is also a common method for querying REST APIs. Following the question mark you need to specify the attribute/field/column, and we already found out above that the API Field Name we are looking for is "county_name". Finally, we are looking for an exact match so we can use the equals sign following by th estring we want to match, "King" (but note you do not want to enclose the string in quotations marks or apostrophes - this is different than how you deal with strings in many programming languages). So our final URL looks like: [`https://data.wa.gov/resource/vtkh-65is.json?county_name=King`](https://data.wa.gov/resource/vtkh-65is.json?county_name=King).

This should get you started on making filtered query API calls. Notice in the screenshot above that there is considerably more documentation on filtering datasets. This example is the tip of the iceberg in what you can do. With SODA it's easy to experiment with different queries directly in your brower window. Here are a couple of things to notice about our example and perhaps you can dig into the documentation to figure out workarounds to the problems that will arise here:

- What happens when you create the filtered query but don't capitalize the 'K' in King?
- What happens when you replace 'json' with 'csv'?
