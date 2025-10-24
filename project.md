---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Data Visualization Project Summary

{(whoami|} 
Name: Alican Kaplan 
City email address: Alican.Kaplan@city.ac.uk
 {|whoami)}

{(task|}

You should complete this datavis project summary document and submit it, along with any necessary supplementary files to **Moodle** by **Sunday 15th December, 5pm UK time**. Submissions will be awarded up to **80 marks** towards your coursework assessment total.

You are also encouraged to regularly commit and push changes to your datavis project throughout the term as you develop your project.

{|task)}

{(questions|}

- How does the annual healthcare expenditure correlate with GP patient satisfaction in the UK?
- What is the relationship between healthcare spending across the UK regions? (Spatial)
- How has the variation in healthcare spending changed across UK regions from 2019 to 2022? (Temporal) 

{|questions)}

{(visualization|}

```elm {l=hidden}
yearlyHealthExpenditureTable : Table
yearlyHealthExpenditureTable =
    """region, 2019-20 (£ Million), 2021-22 (£ Million)
       North East,6958,8955
       North West,19234,25300
       Yorkshire and The Humber,12924,16819
       East Midlands,10690,13962
       West Midlands,14138,18707
       East,13538,18150
       London,26521,33471
       South East,19869,26760
       South West,12811,17095
       England,136682,179219
       Scotland,13702,19035
       Wales, 8025,10697"""
        |> fromCSV


```

^^^elm{m=(tableSummary 12 yearlyHealthExpenditureTable)}^^^

```elm {l=hidden}

yearlyPatientSatisfactionTable : Table
yearlyPatientSatisfactionTable =
    """satisfaction, 2019(%),2020(%),2021(%),2022(%)
       Very good, 45.2, 43.6, 48.2, 37.7
       Fairly good, 37.8, 38.2, 34.7, 34.7
       Neither good nor poor, 10.6, 11.2, 10.4, 14.0
       Fairly poor, 4.4, 4.7, 4.3, 8.1
       Very poor, 2.1, 2.3, 2.4, 5.6"""
        |> fromCSV
    
```

^^^elm{m=(tableSummary 12 yearlyPatientSatisfactionTable)}^^^


```elm {v interactive highlight=13}

firstRegionMap : Spec
firstRegionMap =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewRegions.json"
                [ topojsonFeature "regions" ]

        healthData =
         --  dataFromUrl "https://AlicanK-code.github.io/data/datavisdata.csv"
              dataFromColumns []
                << dataColumn "region" (strColumn "region" yearlyHealthExpenditureTable |> strs)
                << dataColumn "2019-20 (£ Million)" (numColumn "2019-20 (£ Million)" yearlyHealthExpenditureTable |> nums)

        trans = 
            transform
              << lookup "properties.label" (healthData []) "region" (luFields [ "2019-20 (£ Million)" ])
              << lookup "properties.label" (healthData []) "region" (luFields [ "2019-20 (£ Million)" ])         

        enc =
            encoding 
               << tooltips 
                    [ [ tName "properties.label", tTitle "region"]
                    , [ tName "2019-20 (£ Million)", tQuant ]
                    ]
               << color [mName "2019-20 (£ Million)", mQuant, mScale [scScheme "lightmulti"[], scDomain (doNums [ 0, 30000 ])] ]

    in
    toVegaLite
        [ height 400
        , width 300
        , geoData
        , trans []
        , geoshape [ maTooltip ttEncoding ]
        , enc []
        , title "Healthcare Expenditure By Region 2019-20" [] 
       -- , geoshape [ maFill "green", maStroke "white", maTooltip ttEncoding ]
        ]

--, geoshape [ maFill "green", maStroke "white", maTooltip ttEncoding ]
```

```elm { v interactive highlight=13}

secondRegionMap : Spec
secondRegionMap =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewRegions.json"
                [ topojsonFeature "regions" ]

        healthData =
         --  dataFromUrl "https://AlicanK-code.github.io/data/datavisdata.csv"
              dataFromColumns []
                << dataColumn "region" (strColumn "region" yearlyHealthExpenditureTable |> strs)
                << dataColumn "2021-22 (£ Million)" (numColumn "2021-22 (£ Million)" yearlyHealthExpenditureTable |> nums)

        trans = 
            transform
              << lookup "properties.label" (healthData []) "region" (luFields [ "2021-22 (£ Million)" ])
              << lookup "properties.label" (healthData []) "region" (luFields [ "2021-22 (£ Million)" ])         

        enc =
            encoding 
               << tooltips 
                    [ [ tName "properties.label", tTitle "region"]
                    , [ tName "2021-22 (£ Million)", tQuant ]
                    ]
               << color [mName "2021-22 (£ Million)", mQuant, mScale [scScheme "lightmulti"[], scDomain (doNums [ 0, 30000 ])]]

    in
    toVegaLite
        [ height 400
        , width 300
        , geoData
        , trans []
        , geoshape [ maTooltip ttEncoding ]
        , enc []
        , title "Healthcare Expenditure By Region 2021-22" [] 
        ]
```


```elm {v interactive}
barChart : Spec
barChart =
    let
        data =
            dataFromColumns []
                << dataColumn "region" (strColumn "region" yearlyHealthExpenditureTable |> strs)
                << dataColumn "2019-20 (£ Million)" (numColumn "2019-20 (£ Million)" yearlyHealthExpenditureTable |> nums)

        enc =
            encoding
                << position X [ pName "region" ]
                << position Y [ pName "2019-20 (£ Million)", pQuant] 
                << color [ mName "region", mOrdinal, mScale [scScheme "category20" []] ]
                << tooltip [ tName "region", tName "2019-20 (£ Million)"]

    in
    toVegaLite [ data [], enc [],  bar [], title "Healthcare Expenditure 2019-20" [] ]



    -- note to self: add colour, maybe some interaction and a legend?
```

```elm {v interactive}
barChartTwo : Spec
barChartTwo =
    let
        data =
            dataFromColumns []
                << dataColumn "region" (strColumn "region" yearlyHealthExpenditureTable |> strs)
                << dataColumn "2021-22 (£ Million)" (numColumn "2021-22 (£ Million)" yearlyHealthExpenditureTable |> nums)

        enc =
            encoding
                << position X [ pName "region" ]
                << position Y [ pName "2021-22 (£ Million)", pQuant ]
                << color [ mName "region", mOrdinal, mScale [scScheme "category20" []] ]
                << tooltip [ tName "region", tName "2021-22 (£ Million)"]

    in
    toVegaLite [ data [], enc [],  bar [], title "Healthcare Expenditure 2021-22" [] ]

```

```elm {v interactive}
patientSatisfactionOne : Spec
patientSatisfactionOne =
    let
        data =
            dataFromColumns []
                << dataColumn "satisfaction" (strColumn "satisfaction" yearlyPatientSatisfactionTable |> strs)
                << dataColumn "2019(%)" (numColumn "2019(%)" yearlyPatientSatisfactionTable |> nums)

        enc =
            encoding
                << position X [ pName "satisfaction" ]
                << position Y [ pName "2019(%)", pQuant, pAxis [ axGrid False ]]
                << tooltip [ tName "satisfaction", tName "2019(%)"]
                << color [ mName "satisfaction", mScale satisfactionColours, mLegend [] ]


        satisfactionColours =
            categoricalDomainMap
                [ ( "Fairly good", "rgb(153, 255, 153)" )
                , ( "Fairly poor", "rgb(255, 153, 153)" )
                , ( "Neither good nor poor", "rgb(217, 217, 217)" )
                , ( "Very good", "rgb(51, 204, 51)" )
                , ( "Very poor", "rgb(255, 0, 0)" )
                ]
    in
    toVegaLite [ data [], enc [],  bar [], title "Patient Satisfaction 2019" [] ]

```


```elm {v interactive}
patientSatisfactionTwo : Spec
patientSatisfactionTwo =
    let
        data =
            dataFromColumns []
                << dataColumn "satisfaction" (strColumn "satisfaction" yearlyPatientSatisfactionTable |> strs)
                << dataColumn "2022(%)" (numColumn "2022(%)" yearlyPatientSatisfactionTable |> nums)

        enc =
            encoding
                << position X [ pName "satisfaction" ]
                << position Y [ pName "2022(%)", pQuant, pAxis [ axGrid False ]]
                << tooltip [ tName "satisfaction", tName "2022(%)"]
                << color [ mName "satisfaction", mScale satisfactionColours, mLegend [] ]


        satisfactionColours =
            categoricalDomainMap
                [ ( "Fairly good", "rgb(153, 255, 153)" )
                , ( "Fairly poor", "rgb(255, 153, 153)" )
                , ( "Neither good nor poor", "rgb(217, 217, 217)" )
                , ( "Very good", "rgb(51, 204, 51)" )
                , ( "Very poor", "rgb(255, 0, 0)" )
                ]
    in
    toVegaLite [data [], enc [],  bar [], title "Patient Satisfaction 2022" [] ]

```



{|visualization)}

{(insights|}

1. Insight one: 

What is the relationship between healthcare spending across the UK regions? (Spatial)

One key insight that I have identified from my visualizations especially through the use of maps, is that there is higher spending in areas such as London and its surrounding areas, i.e. South East and East. This is seen through London, South East and East having a darker colour when it scales against the variable of £ Millions in their respective regions. We can see that London has had the highest healthcare spending due to the £26,521 million in 2019/20 and has risen to £33,471 million 2021/22. Additionally, we can see that the South East follows this pattern which could be an indication that that they are more populated and therefore have a more expensive health infrastructure. In addition to this, we can see that regardless of this, all regions illustrate an increase in spending from 2019-20 to 2021-22. This likely is a reflection of the COVID-19 pandemic which as we know resulted in more healthcare investment.

Moreover, delving deeper into the Spatial relationship, we can see that there could be Geographic and Economic disparities. This is portrayed as we can see that Northern and rural regions continue to have a lower healthcare spending amount in comparison to more Southern regions. Northern regions i.e. North East has accumulated approximately £6,958 million from 2019-20, and £8,955 million from 2021-22. Thus suggesting that there are potential inequalities in funding or demand for health services.


2. Insight two:

How has the variation in healthcare spending changed across UK regions from 2019 to 2022? (Temporal)

Another key insight that I have identified from my vizualisations is that every region had experienced an increase in healthcare spending over the period 2019-2022. This growth in healthcare spending potentially reflects the increase in demand of healthcare services in order to combat the COVID-19 pandemic. This is illustrated through numerous regions, for example: London 2019-20 : £26,521 million, 2020-21: £33,471 million indicating an approximate ~26% growth in healthcare spending. Additionally, North East 2019-20: £6,958 million, 2021-22: £8,955 million also indicating an approximate ~28% growth in healthcare spending within this region. There were also disparities in growth rate between regions, Wales having a smaller growth in comparison to London and the South East as identified through the darkening colours within the visualization could be an indication that there is a desire to boost healthcare fundings in specific regions. We can definitely say that within this time period, COVID-19 was most likely the key contributing factor in the spending increase in an effort to handle the crisis.

Furthermore, we can also infer that the widespread increase across all regions could suggest a nation effort in order to combat the pandemic across this time period. But, we can also say that the difference in funding per region could be related to reasons such as: differing regional needs or funding priorities. Ultimately, the rise in spending could indicate possible questioning about whether this can be sustained for such a period of time or whether it will dip or change in the future.


3. Insight three:

How does the annual healthcare expenditure correlate with GP patient satisfaction in the UK ?

When discussing my third insight I will be talking about both visualizations, the maps and bar charts. I have discovered that there is a current trend in patient satisfaction for "Very good", as we can see in 2019, people have rated their GP experience as 45.2% and by 2022, that figure drops to 37.7% but is still the highest in terms of the other ratings. There is also a slight dip in "Fairly good", in 2019: 37.8% and in 2022: 34.8% and figures for "Very poor" have more than doubled from 2019: 2.1% to 5.6% in 2022 in addition to "Fairly poor" almost doubling from 4.4% to 8.1%. 

The insight that I can see is that despite the overall healthcare expenditure increasing over the period 2019-22, across all regions, patient satisfaction has declined slightly. This could be due to numerous reasons and factors. For example: An increased pressure on the NHS, GPs due to the ongoing pandemic and diverting resources to combat the pandemic rather than fulfil patient satisfaction. Additionally, even though there was a rise in spending, the decline in patient satisfaction could also be a sign that the spending wasn't enough to satisfy patient experience. We also know that there was a rise in waiting times and a lack of appointments which people were definitely not happy about and could of contributed to the decline in patient satisfaction. Thus, whilst funding played a significant role in addressing healthcare needs, it can be said that it was insufficient to provide enough attention to stop patient experiences from declining.

{|insights)}

{(designJustification|}

1. Colour

One of the main design choices I have implemented for my visualizations is the use of colour. I have used colour throughout all of my visualisations, both maps and bar charts respectively. Using colour has been such a critical design choice within all of my visualizations because it has the ability to illustrate how information is being perceived and understood as well as the additional aesthetically pleasing look it can give to our visualizastions.

When using colour for my first two visualizations on maps, you can see that I have opted to use the lightmulti colour scheme, which is a Sequential Multi-Hue Scheme (https://vega.github.io/vega/docs/schemes/). I have chosen this type of colour scheme because it blends the colours in a smooth progression which is good for the type of data I am representing. Additionally, the use of the Sequential Multi-Hue Scheme for my map visualization is good since it helps understand increase/decreasing changes in my data values, in this case the change in health expenditure overtime and region. We can see this as the regions i.e. London has a darker red in comparison to Wales which is quite green and light illustrating the significance in expenditure. This also is visually appealing and easy to interpret.

Furthermore, when using colour for my bar charts, I have opted for a categorical colour scheme to identify the different regions initially. I did this to have some sort of comparison between my colours and test out what would work. This is because it is effective when highlighting the different regions and since each region is assigned a unique colour, it is easy to identify them.

In addition to this, for my bar chart visualization for patient satisfaction, I have opted to create my own colour scheme using the categoricalDomainMap feature identified in the Bars.md document. I used this as a basis for my colours and then configured it to the colours I wanted to use for my visualization when representation satisfaction. My inspiration for this was the recommended reading: Muth, L-C. (2018) Your friendly guide to colors in data visualization. Datawrapper blog, which gave me the idea to make my own gradient based on this extract, "You want your reader to be able to clearly differentiate between a light green and a …lighter green.". I wanted to be able to portray the differences in responses from Poor to Good in my satisfaction visualization. I used the hex colour editor shown from the third lecture about colours: https://www.w3schools.com/colors/colors_picker.asp?colorhex=03da00 , to create this. I found this very useful and I was satisfied myself with the result.

Ultimately, the use of colour has been very significant and imporant in my visualizations. I have been able to manipulate colour and its various schemes to aide me in answering my research questions and illustrate my visualizations to tell a story and what they are conveying.

2. The use of Maps

Another design choice that I decided to use when constructing my visualizations was the use of Maps, in particularly a map of the UK regions. This main idea stemmed from having a look at the Kensington and Chelsea map visualization in session 3. Once I saw this visualization, I knew that this was something I that I wanted to create a similar visualization. I then continued to look at further resources to see how I would construct such visualization and then I found the geo.md document in the gallery section which gave me examples of different maps and their functions. Using maps is a great foundation to have for my visualizaton because it fits perfectly for the type of data I want to illustrate. This is because, maps are ideal when it comes to visually portraying geographical variations and spatial relationships. We can see this through the variation in healthcare spending across different regions. I.e. London is quite high in spending and a region such as Wales would be much lower, and a map can easily highlight these differences. I also used the topoJSON file shown in session 8 to construct my regions map.

Additionally, along with maps, I'm able to intertwine the use of colours to illustrate the high and lows values geographically making it easier for people to understand my visualization. Moreover, when comparing healthcare spending across a period of time, I'm able to use maps and have them side by side to show changes over time. As well as, to highlight the areas where spending has increased or decreased, resulting in the ability to spot temporal trends relatively easier. I also read upon Choropleth maps from the recommended reading: "Munzner, T. (2015) Chapter 8: Arrange Spatial Data, pp.179-198 in Visualization Analysis and Design, CRC Press", and found that "The major design choices for choropleth maps are how to construct the colourmap, and what region boundaries to use". I would've hoped to use Choropleth maps/Faceting more to analyse my dataset more in-depthly, so this is one thing I would've done next time if I had the chance, as I would've been able to analyse each region individually as opposed to all on one or two singular maps.

Ultimately, the use of maps as a design concept in my visualization has been very significant as it has been able to highlight regional patterns, added spatial and temporal context, enhanced comparisons, and most of all supported me in answering my research questions.

3. Interaction

My final design choice is Interaction. I have decided to use interaction throughout all of my visualizations such as tooltips because they allow me to show extra details within my visualizations. I believe that without interaction, my visualizations would most likely be slightly harder to interpret especially through maps because it wouldn't necessarily show the precise numerical value of how much expenditure is within each region. In addition to this, using a tooltip is useful because it shows information without clutter, helping to create a visually appealing visualization. Furthermore, this encourages user engagement since they are able to hover over a region and be presented with information, in my case, the specified region and its corresponding health expenditure for the year. This ultimately, can lead to users analysing and identifying different patterns moreover, aiding comparison and simplifying complex data present.

In addition to this, I would've liked to implement more interactive features for my visualizations. For example, I wanted to add a drop-down feature to my maps in which I could select different filters based on how much expenditure is spent, I could filter so that I could portray the regions with the lowest amount of expenditure, or the highest for example.
This would've been more significant to illustrate more comparisons within my maps too. I have used the examples from session 6 to illustrate my interaction within my visualizations as well as examples from the interactive.md . 

To summarise, I believe my three design choices: Colour, Map and Interaction are very useful and significant implementations for my research questions and visualizations. This is because all of these design choices intertwine one way or another and they each add a unique element to my visualization to create it.


{|designJustification)}

{(references|}

Maps reference:

Munzner, T. (2015) Chapter 8: Arrange Spatial Data, pp.179-198 in Visualization Analysis and Design, CRC Press

Kensington and Chelsea map visualization

geo.md - Ideas on creating my visualizations

Lecture 8 - https://gicentre.github.io/data/census21/ewRegions.json - topoJSON file to create my map

Colour reference:

https://www.w3schools.com/colors/colors_picker.asp?colorhex=03da00 - Found from the third lecture and used to configure hex colour to build my own colour scheme for bar charts legends.

Muth, L-C. (2018) Your friendly guide to colors in data visualization. Datawrapper blog - This recommended reading material from the third lecture, inspired me to create my own colour scheme.

Bars.md (Bi-direction bar chart section) - This file in the gallery showed me the custom colours via categoricalDomainMap which I used for my bar charts.

https://vega.github.io/vega/docs/schemes/ - Different colour schemes.

Interactive reference:

Session 6

interactive.md

My data:

https://www.ons.gov.uk/peoplepopulationandcommunity/healthandsocialcare/healthcaresystem/articles/measuringnhsexperienceandsatisfactionacrosstheuk/2024-05-30#data-sources-and-quality - Patient Satisfaction, (From ONS Census 2021)

https://gp-patient.co.uk/practices-search - Takes me directly to download patient satisfaction data per year/region/GP

https://www.gov.uk/government/statistics/country-and-regional-analysis-2023 - Healthcare Expenditure per region & overtime. (From ONS Census 2021)

https://github.com/AlicanK-code/data - Any additional data but all of my tables are shown on this document/MD file anyway





{|references)}
