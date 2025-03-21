# Volvo-SAP-R-Visualizations

Welcome to **`Volvo-SAP-R-Visualizations`**, a showcase of advanced data visualizations in RStudio! This project leverages simulated Volvo truck sales, pricing, and delivery data to create four cutting-edge plots, demonstrating my data science skills and passion for enhancing SAP SD processes. Inspired by my goal to secure a Senior SAP SD Solution Consultant role in Sweden, these visualizations bridge my expertise with practical, Volvo-relevant applications.

## Motivation
I built this project to explore how AI-driven insights can enhance SAP SD processes like sales forecasting, pricing optimization, and delivery scheduling. Using RStudio, I transformed mock datasets into interactive and animated visualizations, showcasing trends and relationships that could inform Volvo’s operations. This is part of my broader relocation journey to Sweden, documented in my [`Sweden-Relocation-Playbook`](https://github.com/yourusername/Sweden-Relocation-Playbook).

## Visualizations

### 1. Animated Sales Trend by Truck Model
- **File**: `sales_trend_anim.gif`
- **Data**: `volvo_sales_data.csv` (Order_Date, Quantity, Truck_Model)
- **Description**: An animated line plot showing total sales quantities over time (2023–2025) for Volvo truck models (FH16, VNL, FMX). Lines reveal dynamically by date, highlighting sales patterns.
- **Tools**: `ggplot2`, `gganimate`, `gifski`
- **Relevance**: Supports SAP SD sales forecasting by visualizing demand trends.

### 2. 3D Pricing Scatter Plot
- **File**: `pricing_3d_scatter.html`
- **Data**: `volvo_pricing_data.csv` (Order_Quantity, Competitor_Price, Base_Price, Market_Trend)
- **Description**: An interactive 3D scatter plot mapping order quantity, competitor price, and base price, colored by market trend (high, medium, low). Rotate and zoom to explore pricing dynamics.
- **Tools**: `plotly`
- **Relevance**: Enhances SAP SD pricing optimization with data-driven insights.

### 3. Interactive Delivery Route Map
- **File**: `delivery_route_map.html`
- **Data**: `volvo_delivery_data.csv` (Delivery_Location, Truck_Quantity, Order_ID)
- **Description**: An interactive map plotting delivery locations (Stockholm, Paris, Berlin, Madrid, Depot) with circle sizes proportional to truck quantities. Hover for order details.
- **Tools**: `leaflet`
- **Relevance**: Optimizes SAP SD delivery scheduling and aligns with sustainability goals.

### 4. 3D Interactive Pie Chart
- **File**: `sales_pie_3d.html`
- **Data**: `volvo_sales_data.csv` (Truck_Model, Quantity)
- **Description**: A 3D donut-style pie chart showing the proportion of total sales by truck model. Hover for quantities and percentages, rotate for a dynamic view.
- **Tools**: `plotly`
- **Relevance**: Provides a quick snapshot of sales distribution for SAP SD inventory planning.

## Data Source
The datasets (`volvo_sales_data.csv`, `volvo_pricing_data.csv`, `volvo_delivery_data.csv`) are simulated based on my `Sweden-Relocation-Playbook` projects:
- **Sales**: Daily truck sales (2023–2025).
- **Pricing**: Pricing factors for 1000 records.
- **Delivery**: 50 delivery orders with locations and deadlines.

Generate these files using the Python script(s).

# Load required libraries
library(dplyr)
library(ggplot2)
library(plotly)
library(gganimate)
library(gifski)
library(leaflet)

# Set working directory
setwd("C:/Users/Veteran")

# --- Graph 1: Animated Sales Trend by Truck Model ---
sales_data <- read.csv("volvo_sales_data.csv")
sales_data$Order_Date <- as.Date(sales_data$Order_Date)
sales_summary <- sales_data %>%
  group_by(Order_Date, Truck_Model) %>%
  summarise(Total_Quantity = sum(Quantity), .groups = "drop")
p1 <- ggplot(sales_summary, aes(x = Order_Date, y = Total_Quantity, color = Truck_Model)) +
  geom_line(size = 1) +
  labs(title = "Volvo Truck Sales Trends (2023-2025)", subtitle = "Animated by Date",
       x = "Date", y = "Total Quantity Sold", color = "Truck Model") +
  theme_minimal() +
  scale_color_manual(values = c("Volvo FH16" = "blue", "Volvo VNL" = "red", "Volvo FMX" = "green")) +
  transition_reveal(Order_Date)
anim1 <- animate(p1, nframes = 100, fps = 10, width = 800, height = 600, renderer = gifski_renderer())
print(anim1)
anim_save("sales_trend_anim.gif", anim1)
cat("Animated sales trend saved to:", getwd(), "/sales_trend_anim.gif\n")

# --- Graph 2: 3D Pricing Scatter Plot (from volvo_pricing_data.csv) ---
pricing_data <- read.csv("volvo_pricing_data.csv")
p2 <- plot_ly(pricing_data, x = ~Order_Quantity, y = ~Competitor_Price, z = ~Base_Price,
              color = ~Market_Trend, colors = c("red", "orange", "green"),
              type = "scatter3d", mode = "markers",
              marker = list(size = 5)) %>%
  layout(title = "Volvo Truck Pricing: Quantity vs. Competitor Price vs. Base Price",
         scene = list(xaxis = list(title = "Order Quantity"),
                      yaxis = list(title = "Competitor Price (SEK)"),
                      zaxis = list(title = "Base Price (SEK)")))
print(p2)
htmlwidgets::saveWidget(p2, "pricing_3d_scatter.html")
cat("3D pricing scatter plot saved to:", getwd(), "/pricing_3d_scatter.html\n")

# --- Graph 3: Interactive Delivery Route Map ---
delivery_data <- read.csv("volvo_delivery_data.csv")
location_coords <- data.frame(
  Delivery_Location = c("Stockholm", "Paris", "Berlin", "Madrid", "Depot"),
  Lat = c(59.3293, 48.8566, 52.5200, 40.4168, 57.7089),
  Lon = c(18.0686, 2.3522, 13.4050, -3.7038, 11.9746)
)
delivery_data <- delivery_data %>%
  left_join(location_coords, by = "Delivery_Location")
p3 <- leaflet() %>%
  addTiles() %>%
  addCircles(data = delivery_data, lng = ~Lon, lat = ~Lat, radius = ~Truck_Quantity * 10000,
             popup = ~paste("Order:", Order_ID, "<br>Location:", Delivery_Location, "<br>Quantity:", Truck_Quantity),
             color = "blue", fillOpacity = 0.5) %>%
  addLegend("bottomright", colors = "blue", labels = "Delivery Points", title = "Truck Quantity")
print(p3)
htmlwidgets::saveWidget(p3, "delivery_route_map.html")
cat("Delivery route map saved to:", getwd(), "/delivery_route_map.html\n")

# --- Graph 4: 3D Interactive Pie Chart ---
pie_data <- sales_data %>%
  group_by(Truck_Model) %>%
  summarise(Total_Quantity = sum(Quantity), .groups = "drop")
p4 <- plot_ly(pie_data, labels = ~Truck_Model, values = ~Total_Quantity, type = "pie",
              textinfo = "label+percent", insidetextorientation = "radial",
              marker = list(colors = c("blue", "red", "green")),
              hole = 0.3, scene = "scene") %>%
  layout(title = "Volvo Truck Sales Distribution by Model (2023-2025)",
         scene = list(aspectmode = "cube"), showlegend = TRUE)
print(p4)
htmlwidgets::saveWidget(p4, "sales_pie_3d.html")
cat("3D pie chart saved to:", getwd(), "/sales_pie_3d.html\n")



## Requirements
- **R**: Version 4.3.x or later recommended.
- **R Packages**: `dplyr`, `ggplot2`, `plotly`, `gganimate`, `gifski`, `leaflet`, `htmlwidgets`
- Install with:
  ```R
  install.packages(c("dplyr", "ggplot2", "plotly", "gganimate", "gifski", "leaflet", "htmlwidgets"))

  
