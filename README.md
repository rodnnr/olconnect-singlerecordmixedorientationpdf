## How to insert the correct portrait or landscape PDF page to the corresponding Print Section in Connect Designer

It is assumed here that each record is a multi page with mixed portrait and landscape orientation.
### Datamapper
- Use a Repeat step to loop through each page of the PDF *until no more elements.*
- With the Reapeat step, the "Target Type" of the Go To step should be set to "page"
- Within the Reapeat step, Extract the current page width and\or height values. Example, to extract the current page width:
	> steps.currentPageWidth;
- This creates a detail table containing the width and\or height of each pages. The length of the detail table will be equal to the total number of pages in the record.
### Connect Designer - Control Script
- Add a control script in Connect Designer

        /* this script is for demonstration purposes only.
        This script applies the correct portrait or landscape page background to the correct section.
        It is assumed that each record is multi page with mixed portrait and\or landscape orientation
        */

         let printSections = merge.template.contexts.PRINT.sections;
         let numClones = record.tables.detail.length;

          /* Disable both Sections*/
          printSections.Portrait.enabled = false;
          printSections.Landscape.enabled = false;
          /*for (var i = 0; i < printSections.length; i++) {
          printSections[i].enabled = false;
           }
           */
           /* Get the first section (portrait or landscape) and enable it
           and apply the first page as its background
           */
           let firstSection = record.tables.detail[0].fields.pageWidth > 215 ? printSections.Landscape : printSections.Portrait;
        applyBackground(firstSection,1,1);
        /* Loop through each of the remaining pages, get the page orientation,
        then clone the corresponding section and apply the corresponding page
        as the background of the cloned section
        */
         for(let i=1; i<numClones; i++){
        if(record.tables.detail[i].fields.pageWidth > 215){
            let landscapeClone = printSections.Landscape.clone();
                landscapeClone.name = "landscapeClone_" + i;
                applyBackground(landscapeClone, i+1, i+1);
                firstSection.addAfter(landscapeClone);
        }else{
            let portraitClone = printSections.Portrait.clone();
                portraitClone.name = "portrait_" + i;
                applyBackground(portraitClone, i+1, i+1);
                firstSection.addAfter(portraitClone);
	        }
         }
         function applyBackground(section, start, end){
        section.enabled = true;
        section.background.source = BackgroundResource.DATAMAPPER_PDF;
        section.background.allPages = false;
        section.background.start = start;
        section.background.end = end;
        }
