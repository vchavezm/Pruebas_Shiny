# Pruebas_Shiny
Estudiando Shiny de R
# Load packages ----
library(shiny)
library(quantmod)

# Source helpers ----
source("helpers.R")

# User interface ----
ui <- fluidPage(
  titlePanel("stockVis"),
  
  sidebarLayout(
    sidebarPanel(
      helpText("Select a stock to examine. 
        Information will be collected from Google finance."),
      
      textInput("symb", "Symbol", "SPY"),
      
      dateRangeInput("dates", 
                     "Date range",
                     start = "2013-01-01", 
                     end = as.character(Sys.Date())),
      
      br(),
      br(),
      
      checkboxInput("log", "Plot y axis on log scale", 
                    value = FALSE),
      
      checkboxInput("adjust", 
                    "Adjust prices for inflation", value = FALSE)
    ),
    
    mainPanel(plotOutput("plot"))
  )
)

# Server logic
server <- function(input, output) {

  dataInput <- reactive({
    getSymbols(input$symb, src = "yahoo",
               from = input$dates[1],
               to = input$dates[2],
               auto.assign = FALSE)
  })
  #To fix “Adjust prices for inflation”
  
  #Condition to adjust or not adjust the data
  finalInput <- reactive({
    if (!input$adjust) return(dataInput())
    adjust(dataInput())
  })
  
  output$plot <- renderPlot({
    chartSeries(finalInput(), theme = chartTheme("white"),
                type = "line", log.scale = input$log, TA = NULL)
  })
  
  # output$plot <- renderPlot({   
  #   data <- dataInput()
  #   if (input$adjust) data <- adjust(dataInput())
  #   
  #   chartSeries(data, theme = chartTheme("white"),
  #               type = "line", log.scale = input$log, TA = NULL)
  # })
  #----------------------------------------------------------------------
  
  #----------------------------------------------------------------------
  
  # output$plot <- renderPlot({    
  #   chartSeries(dataInput(), theme = chartTheme("white"),
  #               type = "line", log.scale = input$log, TA = NULL)
  #})
  
  #----------------------------------------------------------------------
  # output$plot <- renderPlot({
  #   data <- getSymbols(input$symb, src = "yahoo",
  #                      from = input$dates[1],
  #                      to = input$dates[2],
  #                      auto.assign = FALSE)
  #   
  #   chartSeries(data, theme = chartTheme("white"),
  #               type = "line", log.scale = input$log, TA = NULL)
  # })
  
  #---------------------------------------------------------------------------------
  # dataInput <- reactive({
  #   getSymbols(input$symb, src = "yahoo", #cambie el "google" por "yahoo"
  #              from = input$dates[1],
  #              to = input$dates[2],
  #              auto.assign = FALSE)
  # })
  # 
  # output$plot <- renderPlot({
  #   
  #   chartSeries(dataInput(), theme = chartTheme("white"), 
  #               type = "line", log.scale = input$log, TA = NULL)
  # })
  #----------------------------------------------------------------------------------
  
}

# Run the app
shinyApp(ui, server)
