# R Shiny applications

[TOC]

Shiny is an R package that makes it easy to build interactive web apps straight from R. You can host standalone apps on a webpage or embed them in R Markdown documents or build dashboards. You can also extend your Shiny apps with CSS themes, htmlwidgets, and JavaScript actions.

## Andersen Lab Shiny Applications

### PCR calculator

An R shiny web app developed for calculating PCR reagents.

* Link to application: [here](https://andersen-lab.shinyapps.io/PCR_calculator/)
* Link to github page with code and explanation of functionality: NA

### HTA Dilutions

An R shiny web app developed to calculate drug dilutions for the high-throughput drug-response assays (sorter or imager). 

* Link to application: [here](https://andersen-lab.shinyapps.io/HTA_dilutions/)
* Link to github page with code and explanation of functionality: [here](https://github.com/AndersenLab/HTA_dilutions)

### Fine-map QTL NILs

An R shiny web app developed to visualize the results from the high-throughput assays (specifically NIL results for fine-mapping a QTL).

* Link to application: [here](https://andersen-lab.shinyapps.io/NIL_genopheno/)
* Link to github page with code and explanation of functionality: [here](https://github.com/katiesevans/finemap_NIL)

### NIL browser

An R shiny web app developed to 1) visualize NIL genotypes and 2) find existing NILs for a project.

* Link to application: [here](https://andersen-lab.shinyapps.io/nil-browser/)
* Link to github page with code and explanation of functionality: NA

### Linkagemapping analysis

An R shiny web app developed to visualize the results from the Andersen Lab linkagemapping experiments in 2014.

* Link to application: [here](https://andersen-lab.shinyapps.io/linkagemapping/)
* Link to github page with code and explanation of functionality: [here](https://github.com/katiesevans/LM-shiny)

---
## How to start a new shiny app?

If you already know R, getting started in shiny just requires learning a few new concepts and some new functions/syntax. Check out this great [tutorial](https://shiny.rstudio.com/tutorial/) to learn more (great video!)!

**Getting started**

You can create a shiny application in Rstudio by clicking `File > New File > Shiny Web App`. Rstudio will install any packages necessary (like `shiny`) and then ask if you want your application to be in one file called `app.R` or two files: `ui.R` and `server.R`. Either way is okay. If you have a large, complex application, it might be easier to split up the UI (user interface) and the server (the meat of the application).

Just like with Rmarkdown, a new Rshiny application comes preloaded with some basic code for you to get a look at. To run a shiny application from Rstudio, simply press the green "run application" button in the upper right of the script panel. A new window should pop up with the application. You can also choose at this point to "open in browser" instead by selecting that button in the upper right of the shiny app.

You now have the beginning of a shiny app! You can add new visual elements to your application in the `ui.R` or in the `shiny::ui()` function in `app.R` and you can create the objects, plots, and manipulate data in the `server`. Here are some of the most commonly used objects (for a full list, check out [this page](https://shiny.rstudio.com/tutorial/written-tutorial/lesson3/)):

**

*Inputs*
You can create an input using one of the following functions in the `ui`, then use that input in the `sever` with `input$input_id`
* `shiny::radioButtons()`
* `shiny::textInput()`
* `shiny::selectInput()`
* `shiny::sliderInput()`
* `shiny::fileInput()`
* `shiny::actionButton()`

*Outputs*
You must create an output in the `server` and then display that output in the `ui`:
| Create ouput in server | Show output in UI |
| --- | --- |
| `shiny::renderPlot()` | `shiny::plotOutput()` |
| `shiny::renderTable()` | `shiny::tableOutput()` |
| `shiny::renderUI()` | `shiny::uiOutput()` |
| `shiny::renderText()` | `shiny::textOutput()` |

**Simple example**

```


```



## Reactive values

XXX

## Publishing your shiny app to shiny.io

XXX