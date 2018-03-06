# Table of Contents

1.  [Action Items](#org1a3aa75)
    1.  [Prep](#orgf1c38c3)
        1.  [Review next steps for SageMaker in documentation](#org927f050):jack:manish:
        2.  [Push to Github](#org6ad9a1b):jack:
        3.  [Upload to S3](#org9f787af):manish:
    2.  [Tech](#orga32a129)
        1.  [Run through the process with the data subset (~6000 points)](#org013cc99)
        2.  [Load the model in Deeplens](#org4fffed5)
        3.  [Prep ALL the data (.lst, and upload to S3)](#orgbf0ad73)
        4.  [Make SageMaker model with full data set](#orgdab40b1)
        5.  [Deploy model to Deeplens](#orgf6e644e)
    3.  [Trees](#org1ccdc0f)
        1.  [Identify the species for some for some life samples](#org40f2271)
    4.  [Test Deeplens+Model on real leaves](#orgbe25ed3)
2.  [plant photo data set](#orgb2216b5)



<a id="org1a3aa75"></a>

# Action Items


<a id="orgf1c38c3"></a>

## Prep


<a id="org927f050"></a>

### TODO Review next steps for SageMaker in documentation     :jack:manish:


<a id="org6ad9a1b"></a>

### TODO Push to Github     :jack:


<a id="org9f787af"></a>

### TODO Upload to S3     :manish:


<a id="orga32a129"></a>

## Tech


<a id="org013cc99"></a>

### TODO Run through the process with the data subset (~6000 points)


<a id="org4fffed5"></a>

### TODO Load the model in Deeplens


<a id="orgbf0ad73"></a>

### TODO Prep ALL the data (.lst, and upload to S3)


<a id="orgdab40b1"></a>

### TODO Make SageMaker model with full data set


<a id="orgf6e644e"></a>

### TODO Deploy model to Deeplens


<a id="org1ccdc0f"></a>

## Trees


<a id="org40f2271"></a>

### TODO Identify the species for some for some life samples


<a id="orgbe25ed3"></a>

## Test Deeplens+Model on real leaves


<a id="orgb2216b5"></a>

# plant photo data set

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">&#xa0;</th>
<th scope="col" class="org-left">idea 1</th>
<th scope="col" class="org-left">idea 2</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">initial</td>
<td class="org-left">20GB</td>
<td class="org-left">44M</td>
</tr>


<tr>
<td class="org-left">unzipped</td>
<td class="org-left">22GB</td>
<td class="org-left">350M</td>
</tr>


<tr>
<td class="org-left">images</td>
<td class="org-left">included</td>
<td class="org-left">download</td>
</tr>


<tr>
<td class="org-left">data point</td>
<td class="org-left">jpg + xml</td>
<td class="org-left">a line in CSV (but need to download the corresponding image)</td>
</tr>


<tr>
<td class="org-left">number of data points</td>
<td class="org-left">0.25 mil</td>
<td class="org-left">1.1+ mil</td>
</tr>


<tr>
<td class="org-left">image size</td>
<td class="org-left">~100K</td>
<td class="org-left">~500K</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">effort needed</td>
<td class="org-left">need code to parse the data points to produce .lst file and organize corresponding images</td>
<td class="org-left">need code to take a subset of points, download and organize corresponding images and produce a .lst file</td>
</tr>
</tbody>

<tbody>
<tr>
<td class="org-left">subset size</td>
<td class="org-left">200</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">&#xa0;</td>
</tr>
</tbody>
</table>

    # <Image>
    # 	<FileName>248738.jpg</FileName>
    # 	<Species>Justicia nyassana Lindau</Species>
    # 	<Origin>eol</Origin>
    # 	<Author>mark hyde, bart wursten and petra ballings, bart wursten, flora of zimbabwe</Author>
    # 	<Content></Content>
    # 	<Genus>Justicia</Genus>
    # 	<Family>Acanthaceae</Family>
    # 	<ObservationId>174381</ObservationId>
    # 	<MediaId>248738</MediaId>
    # 	<YearInCLEF>PlantCLEF2017</YearInCLEF>
    # 	<LearnTag>Train</LearnTag>
    # 	<ClassId>2365</ClassId>
    # </Image>

    # https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification.html

    # A .lst file is a tab-separated file with three columns that contains
    # a list of image files. The first column specifies the image index,
    # the second column specifies the class label index for the image, and
    # the third column specifies the relative path of the image file. The
    # image index in the first column must be unique across all of the
    # images.

    from bs4 import BeautifulSoup
    import os
    import os.path

    lst_filename = "all.lst"
    lst_contents = []
    index = 1

    def parseme(f):
        r = open(f).read()
        x = BeautifulSoup(r, features="xml")
        fn = x.Image.FileName.contents[0]
        klass = x.Image.ClassId.contents[0]
        return [fn, klass]


    for root, dirs, files in os.walk("."):
        for f in files:
            extension = os.path.splitext(f)[1]
            if extension == ".xml":
                fn, klass = parseme(f)
                lst_contents.append(str(str(index) + "\t" + klass + "\t" + "./" + fn + "\n"))
                index += 1

    with open(lst_filename, "w") as f:
        for line in lst_contents:
            f.write(line)
