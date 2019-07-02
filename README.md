# BadMedicine.Dicom
The purpose of BadMedicine.Dicom is to generate large volumes of complex (in terms of tags) dicom images for integration/stress testing ETL and image management tools.

There are a number of public sources of Dicom clinical images e.g. [TCIA ](https://www.cancerimagingarchive.net/).  The difficulty with using these for integration/stress testing is that they often:

- Are anonymised (majority of tags have been removed)
- Do not represent the breadth of Modalities/Tags found in a live clinical [PACS](https://en.wikipedia.org/wiki/Picture_archiving_and_communication_system).
- Take up a lot of space

BadMedicine.Dicom generates dicom images on demand based on an anonymous aggregate model of tag data found in scottish medical imaging.  It is an extension of [BadMedicine](https://github.com/HicServices/BadMedicine) which generates traditional EHR records.

## Usage

BadDicom is available as a [nuget package](https://www.nuget.org/packages/HIC.BadMedicine.Dicom/) for linking as a library

The standalone CLI (BadDicom.exe) is available in the [releases section of Github](https://github.com/HicServices/BadMedicine.Dicom/releases)

Usage is as follows:

```
BadDicom.exe c:\temp\testdicoms
```
_Generates 10 dicom studies (~700MB)_

```
BadDicom.exe c:\temp\testdicoms 5 10 --NoPixels
```
_Generates 10 dicom studies from a pool of 5 patients without pixel data (~3MB)_

You can pass `-s` to seed the random number generators.  Seeding will ensure the same StudyDate, PatientId etc get generated but will __not__ affect UIDs generated (UIDs are always unique)

```
BadDicom.exe c:\temp\testdicoms 5 10 --NoPixels -s 100
```

## EHR Datasets

If you want to generate EHR datasets with a shared patient pool with the dicom data (e.g. for doing linkage) you can provided the -s (seed) and use the main [BadMedicine](https://github.com/HicServices/BadMedicine) application.

```
BadDicom.exe c:\temp\testdicoms 12 10 -s 100
BadMedicine.exe c:\temp\testdicoms 12 100 -s 100
```
_Generates a pool of 12 patients and 10 Studies (for random patients) then 100 rows of data for each EHR dataset_

# Library Usage
You can generate test data for your program yourself by referencing the [nuget package](https://www.nuget.org/packages/HIC.BadMedicine.Dicom/):

```csharp
//create a test person
var r = new Random(23);
var person = new Person(r);

//create a generator 
var generator = new DicomDataGenerator(r,null,"CT");
            
//create a dataset in memory
DicomDataset dataset = generator.GenerateTestDataset(person);

//values should match the patient details
Assert.AreEqual(person.CHI,dataset.GetValue<string>(DicomTag.PatientID,0));
Assert.GreaterOrEqual(dataset.GetValue<DateTime>(DicomTag.StudyDate,0),person.DateOfBirth);
```

## Building

You can build a OS specific binary

First build BadDicom.csproj
```
dotnet publish BadDicom.csproj -r win-x64 --self-contained
cd .\bin\Debug\netcoreapp2.2\win-x64\
```

# Tag Data

Basic random patient information (age, CHI etc) are generated by [BadMedicine](https://github.com/HicServices/BadMedicine)

The following tags are populated in dicom files generated:

|Tag | Model |
|-----|-----|
| PatientID | The CHI number of the (random) patient|
| StudyDate | A random date during the patients lifetime |
| StudyTime | Random time of day with hours favoured during the middle of the working day.|
| SeriesDate | Same as StudyDate* |
| PatientAge | Age of patient at SeriesDate e.g. "032Y"|
| Modality | Random modality for which we have at least 1 image locally. (proportionate to modality popularity)|
| StudyDescription | Random description that exists in the modality (proportionate to frequency seen) |

*SeriesDate is always the same as Study Date (see `Seres` constructor), for secondary capture this should/could not be the case (we should look at how this corresponds in the PACS data we have)

# Pixel Data
Currently pixel data is written as a black square with the SOP Instance UID written in white.

