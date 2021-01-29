OS Downloads API: Tutorial - DRAFT {#os-downloads-api-tutorial---draft .list-paragraph}
==================================

The [OS Downloads API](https://osdatahub.os.uk/docs/downloads/overview)
is a service that allows users to automate the discovery and download of
OS OpenData. This tutorial will demonstrate how a simple csv file can be
used alongside tools such as a python script or FME workbench, to
utilise the automated download feature, and make sure you always have
the most up-to-date data on your servers. A script or workbench can be
run daily/weekly/monthly using a batch file and gives users the peace of
mind that they are always using the very latest datasets. Downloads will
only occur if an update is available therefore making the whole process
as efficient as possible.

The tutorial will discuss the methodology involved in downloading
available OS products, as well as providing examples in the format of a
python script and FME workbench. Please feel free to take a look at the
examples, have a play and then customise them for your own specific
needs. **\<link to github\>?**

![](./media/image1.png){width="6.6930555555555555in"
height="2.0701388888888888in"}

*Example setup*

CSV
---

Firstly, we wanted a method of recording the status of our current
datasets and that would also be quick and easy to process within
different tools -- arriving at a simple csv format. The csv file stores
the latest version date of your datasets which are then compared against
the latest product version dates.

All the user has to do is:

-   record which datasets they wish to download (id)

-   the date that they last downloaded the dataset (user\_version)

-   which format they require the data to be in (user\_format), ranging
    from CSV to GeoPackages

-   the area that the data needs to cover (user\_area). The majority of
    the datasets can be downloaded with full GB coverage, however, there
    are currently four datasets where individual OS tiles can be
    selected.

![](./media/image2.png){width="4.625in"
height="2.8890748031496063in"}

*Example version of the csv file*

Once the user has initially setup the csv file, it does not need to be
edited again unless a new dataset is required, or they want to change
the data format/area.

Methodology
-----------

The attraction of the process that we will describe is its scalability
-- as and when new products arrive or old are retired on the OS
Downloads API, all the user has to do is update their csv file. All the
required information for a download is obtained via different calls to
the API (handled by a script or workbench), which will be highlighted
below.

It is worth noting that due to differences between various datasets, the
parameters can be different when it comes to downloading data. The
process below helps capture all the different permutations of various
requests which helps highlight the scalability credentials.

*Step 1: Products*

We need to obtain the latest product information by firing off a request
to this url: <https://api.os.uk/downloads/v1/products>

The request returns data in a JSON format that contains useful
information about all the products available for download, including;
id, name, description, version and url.

![](./media/image3.png){width="6.6930555555555555in"
height="1.8993055555555556in"}

*Sample of JSON result regarding products*

*Step 2: Version Dates*

We need to determine which parts of this request are required, and that
is achieved by joining the results to a copy of our CSV file (joining on
id). This will keep only the information required based on which OS
products are listed in the file. In the same step, we can also format
the 'version' dates (from text to date) so we can compare and determine
if any updates are available.

![](./media/image4.png){width="6.6930555555555555in"
height="0.4215277777777778in"}

*Example of join*

*Step 3: Product URL*

If an update is available, we need to fire off another URL request, this
time using the 'url' column information. This url will provide more
detailed information about the specific product, such as the
'downloadsUrl' and what formats and area coverage are available. This
information will be used to build our final request to receive the
download file. An example of this request (results in JSON format):
<https://api.os.uk/downloads/v1/products/OpenGreenspace>

![](./media/image5.png){width="6.6930555555555555in"
height="2.5729166666666665in"}

*All available parameters for downloading OS Open Greenspace*

*Step 4: DownloadsUrl*

Every available dataset requires a unique URL in order to be downloaded,
so using the above example, a URL is required for each individual area
(e.g. HP, HT, HU etc). Within our python script and FME workbench, we
create a list of all the required areas as specified in the CSV file and
start to build our next request.

We take the 'downloadsUrl' obtained in our previous request and add to
it the format we require the data (specified in CSV file) and complete
it with a specific area (from list created above). So, if I wanted OS
Open Green space, in GML format, for all of GB, then my request would
look like:

<https://api.os.uk/downloads/v1/products/OpenGreenspace/downloads?format=GML&area=GB>

However, if I wanted the same product in shapefile format for areas HP
and HT, then I would require two separate requests that look like:

HP:
<https://api.os.uk/downloads/v1/products/OpenGreenspace/downloads?format=ESRI%C2%AE%20Shapefile&area=HP>

HT:

<https://api.os.uk/downloads/v1/products/OpenGreenspace/downloads?format=ESRI%C2%AE%20Shapefile&area=HT>

*Step 5: Product Download*

We are almost there! Using the URL's we have created in step 4, we can
now fire off the requests that allow us to obtain the final download
link (see 'url' in screenshot below) and the 'fileName' which allows us
to determine the download format e.g. zip, csv, vector tiles etc, and
also use to name the downloaded file.

![](./media/image6.png){width="6.6930555555555555in"
height="0.5222222222222223in"}

*The 'url' is our download link, confirmed by the presence of
'&redirect' at the end of the request*

Note:

There are some datasets that contain multiple files for the same area,
for example, eleven CSV files are available to download for GB coverage
related to the OS Open Linked Identifiers dataset. Within the python
script and FME workbench, a list is created that loops through firing
off their individual download URL.

*6: CSV Update*

Once all the downloads have been processed, the latest version dates are
recorded and exported back out to the csv file so they can be used for
future comparisons and allow users to check which version they currently
have.

And that is it -- all required datasets have been downloaded!

Software/Tools
--------------

Feel free to test the methodology by downloading the python script or
FME workbench. The python script was put together using the
[pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/install.html)
software library and users will need to install this in order for the
script to run. It is also worth noting that no error catching currently
exists which might be worth implementing e.g. no/lost connection to API,
misspelt names in csv etc. The workbench was created and tested using
FME Desktop 2019.

![](./media/image7.png){width="6.6930555555555555in"
height="3.8402777777777777in"}

*Sample of python code*

![](./media/image8.png){width="6.6930555555555555in"
height="6.317361111111111in"}

*FME Workbench*

You are also not limited to python or FME -- experiment with other
software or tools that you are familiar with and replicate the
methodology. For a bit of inspiration, see how Node JS has been used to
download Terrain50 data here:
<https://labs.os.uk/public/os-data-hub-tutorials/web-development/automated-open-data-download>
