# SurveyJS-Note
List out all investigation topic that are not official documentation on SurveyJS. 
The SurveyJS support team is great to support but no fully anwser on the question that we ask.

## PDFExport: How to render the invisible question which is hidden by logic?
####
>Problem: 
>- I have 3 questions: A, B and C.
>- Question B is invisibled by the visibleIf attribute.
>- So when I use PDF export, I only see question A and C.
>- Can PDF-Export has the option to also render the question B in the PDF file?
>- https://surveyjs.answerdesk.io/ticket/details/t9070/pdf-export-how-to-render-the-invisibled-question

####
>Solution: 
 - There is only one solution for this one is "remove the visibleIf" condition on the question before trigger the PDF export.
 ```
 json.pages.flatMap(page =>  page.elements).forEach(e => {
     e.visibleIf = '1';
});
 ```
 - So the next side affect after removing the visibleIf condition is that:
 - In PDF file, How we can know the question is invisibled by the condition? so that you can change the question color (to gray color example) or add new description like ***This question was not displayed to the respondent, due to certain criteria in previous questions.***?
 - To solve this issue, we must add all question conditions to the map:
```
var map = new Map();
json.pages.flatMap(page =>  page.elements).forEach(e => {
    map.set(e.name, e.visibleIf);
     e.visibleIf = '1';
});
```
 - And then run the expression manually:
```
...
surveyPDF
        .onRenderQuestion
        .add(function (survey, options) {
          var visibleIf = true;
          if(options.question.conditionRunner){
            const v = map.get(options.question.name);
            if(v){
                options.question.conditionRunner.expression = v;
                visibleIf =  options.question.conditionRunner.run(data);
            }
          }
          if(options.question.conditionRunner &&  !visibleIf){
            // change invisible question color and options color
            ...
          }
...
```




## PDFExport: How to add the cover page?
####
>Problem: 
>- The customer ask if we can have the cover page in exported PDF file. The cover page is the first page in PDF with text ***User Assessment Response*** and some logo/image of company trademark.

####
>Solution:
> There is no way to do that with PDF export. But you can do it by adding one new SurveyJS page with HTML question and Image questions. So that you apend that page to the SurveyJS model before trigger the PDF export.
```
{
...
"pages":[
{
            "name": "cover",
            "_index": 0,
            "elements": [
                {
                    "type": "html",
                    "name": "cover-padding",
                    "indent": 5,
                    "html": "<p style='padding-top:50px'></p>"
                },
                {
                    "type": "image",
                    "indent": 5,
                    "name": "cover-banner",
                    "imageLink": "http://localhost:8000/img/logo.png",
                    "imageWidth": "600px",
                    "imageHeight": "200px"
                },
                {
                    "type": "html",
                    "name": "cover-company",
                    "indent": 5,
                    "html": "<h1 style='color:black'>User response for assessment</h1>\n<h1 style='color:black'>&lt;Example Company&gt;</h1>"
                },
                {
                    "type": "html",
                    "name": "cover-padding-2",
                    "indent": 5,
                    "html": "<p style='padding-top:350px'></p>"
                },
                {
                    "type": "image",
                    "name": "cover-bottom-image",
                    "indent": 5,
                    "imageLink": "http://localhost:8000/img/logo.png",
                    "imageWidth": "300px",
                    "imageHeight": "75px"
                }
            ]
        },
...
],
...

}
```

## PDFExport: How to add more text/paraprap on question?
####
>Problem: 
> PDF render the question as what we see in SurveyJS. But the customer can ask to to change the color of question in PDF when it is invisibled by condition.

####
>Solution:
> By using API `createHTMLFlat` you can create the HTML element in that question:

```
 surveyPDF
        .onRenderQuestion
        .add(function (survey, options) {
          if(options.question.conditionRunner &&  !visibleIf){
                // change invisible question color and options color
                for (const [key, value] of Object.entries(options.bricks)) {
                    var plainBrickUnfold = value.unfold();
                    for (const [key, plainBrickUnfoldItem] of Object.entries(plainBrickUnfold)) {
                        plainBrickUnfoldItem.textColor = "#959595";
                      }
                }
                const extraTitle = '<i style="color:#959595;font-size:11pt;font-family: Open Sans">This question was not displayed to the respondent, due to certain criteria in previous questions.</i>';
                var plainBricks = options.bricks[0].unfold();  
                var lastBrick = plainBricks[plainBricks.length - 1];
                var point = SurveyPDF.SurveyHelper.createPoint(lastBrick);
                return new Promise(function (resolve) {
                    SurveyPDF
                        .SurveyHelper
                        .createHTMLFlat(point, options.question, options.controller, extraTitle)
                        .then(function (descBrick) {
                          options.bricks.push(descBrick);
                          resolve();
                         });
               ....
                  
          
          
        }

```

## PDFExport: How to delete all question options?
####
>Problem: 
>The customer ask to remove all options of the invisible question in PDF file. How to do that?

####
>Solution: 
>By using debug and API releated to `brick`:

```
 surveyPDF
        .onRenderQuestion
        .add(function (survey, options) {
        
          var plainBricks = options.bricks[0].unfold();
          
          //remove all matrix column text
          const lengthFirstBrick =options.bricks[0].bricks.length;
          for(var i =2; i<lengthFirstBrick;i++){
              options.bricks[0].bricks.pop();
          }
          plainBricks = options.bricks[0].unfold();
          
          // remove all options
          const length = options.bricks.length;
          for(var i = 1; i<length;i++){
              options.bricks.pop();
          }

```

## PDFExport: How to change the text for file upload question?
####
>Problem: 
>The customer ask to change the text for file upload question. When the file question has answer, use ***User has uploaded documents.*** otherwise use ***No documents uploaded by user***

####
>Solution: 
>By using `createHTMLFlat` API:

```
 surveyPDF
        .onRenderQuestion
        .add(function (survey, options) {

            if(options.question.getType() =='file'){
                const hasFileText = '<h1 style="font-size:12pt;font-family: Open Sans;font-weight: normal;color: rgb(75,75,75)">User has uploaded documents</h1>';
                const noFileText = '<span style="font-size:12pt;font-family: Open Sans;font-weight: normal;color: rgb(75,75,75)">No documents uploaded by user</span>';
                const allValues = options.question.getAllValues();
                var plainBricks = options
                .bricks[0]
                .unfold();
                const hasAnswer = allValues[options.question.name];
                const text = hasAnswer? hasFileText: noFileText;
                

                // delete the last brick that is the underlink
                options.bricks[0].bricks[2].bricks.pop();
                plainBricks = options.bricks[0].unfold();

                //add the line
                var lastBrick = plainBricks[plainBricks.length - 1];
                var point = SurveyPDF.SurveyHelper.createPoint(lastBrick);
                return new Promise(function (resolve) {
                    SurveyPDF
                        .SurveyHelper
                        .createHTMLFlat(point, options.question, options.controller, text)
                        .then(function (descBrick) {
                            const descBrickUnFold = descBrick.unfold();
                            options.bricks.push(descBrick);
                            resolve();
                        })
                        
                        ;
                });
                
               
            }

```
