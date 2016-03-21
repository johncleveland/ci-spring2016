Interactive web applications in R
=================================

This lesson presents an introduction to creating interactive web applications using the Shiny package in R. We will cover the following concepts:

-   [file structure template](#app-template)
-   [input objects](#input-objects)
-   [output objects](#output-objects)
-   [create a reactive object](#create-reactive-object)
-   [customizing layout design](#design-layout)
-   [upload or download files](#download-button)
-   [deploying your app](#how-to-deploy)

Make sure the shiny package is installed and loaded

``` r
# library(shiny)
```

App template
------------

A shiny app consists of two components:

-   **user interface** which defines what users will see in the app
-   **server** which defines the instructions for assembling components of the app as a function of input and output objects

There are two ways to structure an app:

1.  Create one file called `app.R`. In this file, define objects `ui` and `server` with the assignment operator `<-` and then pass them to the function `shinyApp()`.
2.  Split the template into two files named `ui.R` and `server.R`.

We will use the second option here to make an app. When the `shiny` package is installed and loaded, RStudio will identify this file structure and create a green arrow with a "Run App" button when you open a file in the app. Applications use your current R session and are displayed in a browser window.

Test this by looking at an example within the shiny package:

``` r
runExample("01_hello")
```

Notice the stop sign that appears in the Console window while your app is running. This is because it is using the current R session. Make sure to end the app by either closing the external window or clicking the stop sign.

Create a new folder in your current working directory with ui and server files, and a copy of the data folder. Read in the data in the server file and display a title in the user interface file

``` r
# ui.R

fluidPage(
  
  titlePanel("Hello Shiny!")
  
)
```

``` r
# server.R

# Read in data
plots <- read.csv("Data/plots.csv", stringsAsFactors = FALSE)
species <- read.csv("Data/species.csv", stringsAsFactors = FALSE)
surveys <- read.csv("Data/surveys.csv", na.strings = "", stringsAsFactors = FALSE)
 
function(input, output){
  
}
```

Input objects
-------------

The user interface and the server file interact with each other through \*\*\_\_Input()\*\* and \*\*\_\_Output()\*\* functions.

Input objects collect information from the user and saves them in a list. The first two arguments for each input widget are `inputId =` which gives a name to the input object, and `label =` which is displayed to the user. Other arguments depend on the type of input widget. Input objects are stored in a list and are therefore referred to in the server file as `input$inputID`. A gallery of input widgets can be found on the RStudio website [here](http://shiny.rstudio.com/gallery/widget-gallery.html).

We will add an input object to select one of the four taxa in the portals data set. Use the `selectInput()` function to create an input object called `pick_taxa` in the ui file. Use the `choices =` argument to define a vector with the options.

``` r
# ui.R

fluidPage(
  
  titlePanel("Hello Shiny!"),

  selectInput("pick_taxa", label = "Pick a taxa", choices = unique(species$taxa))  

)
```

Input objects are **reactive** which means that an update to this value by a user notifies the server file that its value has been changed.

Choices for inputs can be named using a list to match the display name to the value such as `list("Male" = "M", "Female" = "F")`. [Selectize](http://shiny.rstudio.com/gallery/selectize-vs-select.html) inputs are a useful option for long drop down lists. Always be aware of what the default value is for input objects you create.

Output objects
--------------

Output objects are created through the combination of pairs of `render*()` and `*Output()` functions. Define a list of output objects in the server file using render functions:

``` r
output$my_plot <- renderPlot({})
```

Render functions build an object to display in the server file. Use curly brackets inside parenthesis if there are input objects being used. The code inside the body of the render function will run whenever a reactive value inside the code changes. Use the output ID to refer to that output element in the user interface file to place it in the app.

``` r
ui.R
fluidPage(
  plotOutput("my_plot")
  )
```

| render function   | output function      | displays              |
|-------------------|----------------------|-----------------------|
| renderPlot()      | plotOutput()         | plots                 |
| renderPrint()     | verbatimTextOutput() | text                  |
| renderText()      | textOutput()         | text                  |
| renderTable()     | tableOutput()        | static table          |
| renderDataTable() | dataTableOutput()    | interactive table     |
| renderUI()        | uiOutput()           | reactive input object |

Use the `renderPlot()` function to define a plot showing the total number of observations per year of the taxa selected by the user.

``` r
  output$taxa_plot <- renderPlot({
    taxa_subset <- filter(species, taxa == input$pick_taxa)$species_id
    surveys_subset <- filter(surveys, species_id %in% taxa_subset)
    barplot(table(surveys_subset$year))
  })
```

Use the corresponding `plotOutput()` function in the ui file to display the plot. Make sure the name of the plot output object is in quotes and that ui elements are separated by commas.

``` r
plotOutput("taxa_plot")
```

Note that you can change the size of plot outputs by defining the number of pixels.

### Exercise

Exercise: Add an input widget to control the survey months included in the plot.

``` r
# in ui.R
sliderInput("pick_months", "Select which months to include", min = 1, max = 12, value = c(1,12)),
plotOutput("taxa_plot_2")

# in server.R
output$taxa_plot_2 <- renderPlot({
  taxa_subset <- filter(species, taxa == input$pick_taxa)$species_id
  surveys_subset <- filter(surveys, species_id %in% taxa_subset & month %in% input$pick_months[1]:input$pick_months[2])
  barplot(table(surveys_subset$year))
  })
```

Design layout
-------------

Within the user interface, you can arrange where elements appear by defining a page layout. The `fluidPage()` consists of rows which contain columns of elements. Define the width relative to a 12-unit grid of each element within each column using the function `fluidRow()` and listing columns in units of 12. For each column first define its width. The argument `offset =` can be used to add extra spacing. Notice how the elements shift when the browser window is re-sized.

``` r
# ui.R

fluidPage(
  fluidRow(
    column(4,
           "4"
    ),
    column(4, offset = 4,
           "4 offset 4"
    )      
  ),
  fluidRow(
    column(3, offset = 3,
           "3 offset 3"
    ),
    column(3, offset = 3,
           "3 offset 3"
    )  
  ))
```

You can organize elements using higher level pre-defined layouts such as `sidebarLayout()`, `splitLayout()`, or `verticalLayout()`. Elements can be layered on top of each other using `tabsetPanel()`, `navlistPanel()`, or `navbarPage()`.

Organize your application using the [sidebar layout](http://shiny.rstudio.com/reference/shiny/latest/sidebarLayout.html). Place the input widgets in the sidebar and the plot in the main panel. *Think about where you need to put commas and parentheses!*

``` r
  sidebarLayout(
    sidebarPanel(
      selectInput("pick_taxa", label = "Pick a taxa", choices = unique(species$taxa)),
      sliderInput("pick_months", "Select which months to include", min = 1, max = 12, value = c(1,12))
      ),
    mainPanel(
      # plotOutput("taxa_plot"),
      plotOutput("taxa_plot_2")
    )
  )
```

Use the argument `position = "right"` in the `sidebarLayout()` function if you prefer to have the side panel appear on the right.

Make the main panel a [tabset panel](http://shiny.rstudio.com/articles/tabsets.html) with a first tab titled "Plot" and a second titled "Data" which displays all of the survey data used in the plot as a data table ouput. Use the `tabsetPanel()` function with a list of tabs created with `tabPanel()`. Make sure to account for all of your parentheses!

``` r
# in server
  output$surveys_subset <- renderDataTable({
    taxa_subset <- filter(species, taxa == input$pick_taxa)$species_id
    surveys_subset <- filter(surveys, species_id %in% taxa_subset & month %in% input$pick_months[1]:input$pick_months[2])
    return(surveys_subset)
  })

# in ui
mainPanel(
      tabsetPanel(
        tabPanel("Plot",
                plotOutput("taxa_plot_2")),
        tabPanel("Data", 
                dataTableOutput("surveys_subset"))
      )
    )
```

In addition to titles for tabs, you can also use fun [icons](http://shiny.rstudio.com/reference/shiny/latest/icon.html).

### Customize appearance

Along with widgets and output objects, you can add headers, text, images, links, and other html objects to the user interface. There are shiny function equivalents for many common html tags such as `h1()` through `h6()` for headers. You can use the console command line to see that the return from these functions produce HTML code.

Add a bold level 3 header above the plot in the main panel that includes the input object `pick_taxa`. Hint: render a text object in the server file.

``` r
# in server
  output$title_text <- renderText({
    paste0("Total number of ", input$pick_taxa, "s observed by year in months ", input$pick_months[1], " through ", input$pick_months[2])
  })

# in ui
h3(strong(textOutput("title_text")))
```

See [here](http://shiny.rstudio.com/articles/tag-glossary.html) for additional html tags you can use.

For large blocks of text consider saving the text in a separate markdown, html, or text file and use an `include` function ([example](http://shiny.rstudio.com/gallery/including-html-text-and-markdown-files.html)).

Add images by saving those files in a folder called **www**. Link to it with img(src="<file name>")

Bonus Exercise: Below the graph, add text that says what year had the maximum number of the selected taxa recorded.

``` r
  output$max_year_text <- renderText({
    max_year <- names(tail(sort(table(surveys_subset()$year)),1))
    paste0("The most ", input$pick_taxa, "s were recorded in ",max_year)
  })
```

Reactive objects
----------------

Input objects that are used in multiple render functions to create different output objects can be created independently as **reactive** objects. This value is then cached to reduce computation required, since only the code to create this object is re-run when input values are updated. Use the function `reactive()` to create reactive objects and use them with function syntax, i.e. with `()`. Reactive objects are not output objects so do not use `output$` in front of their name.

We will make the filtered data set a reactive object called `surveys_subset` to use to render both the plot and the data table, instead of repeating the code to create the filtered data set. Creating this object needs to happen in the server file preceeding its use.

``` r
# in server
  surveys_subset <- reactive({
    taxa_subset <- filter(species, taxa == input$pick_taxa)$species_id
    # taxa_subset <- subset(species, taxa == input$pick_taxa, select = species_id) # if dplyr not loaded
    surveys_subset <- filter(surveys, species_id %in% taxa_subset)
    return(surveys_subset)
  })
```

To use `surveys_subset` in render functions, refer to it with function syntax.

``` r
# in server
  output$taxa_plot <- renderPlot({
    barplot(table(surveys_subset()$year))
  })

  output$surveys_subset <- renderDataTable({
    surveys_subset()
  })
```

Upload or download files
------------------------

Add a button to download a csv of the data or an image of the graph

Deploy your app
---------------

Shiny extensions
----------------

There are many ways to extend the functionality and sophistication of Shiny apps using existing tools and platforms.

Leaflet

Interactive graphics

Plot.ly

Additional references
---------------------