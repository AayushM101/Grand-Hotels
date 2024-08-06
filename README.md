#  Grand Hotels Revenue Analysis Dashboard


## Problem Statement

Atliq Grand, a renowned 5-star hotel chain across India, is facing a decline in market share amidst growing competition. To address this challenge, the Operations Manager, Mr. Gaurav Dev, seeks to analyze the revenue performance of the hotels. The objective is to identify key areas for improvement to enhance revenue generation and reduce operational expenditure. This analysis will provide insights necessary for strategic decision-making to regain market position and ensure sustainable growth.

## Recommeneded Metrics
        1. Revpar(Revenue Per available Room) = Total Revenue/Total Room available to sell
        2. ADR(Average daily Rate) = Total rooms Revenue/Number of rooms sold 
        3. Occupancy% =(Total Rooms Revenue/Total Rooms available)*100
        **Note Revpar = ADR*Occupancy
        4. Revenue
        5. DSRN = Total Capacity/Number of Days
        6. Realisation = DURN/DBRN 
        7. DBRN = Total room Bookings/Number of Days
        8. DURN = Total checkedout rooms/Number of Days
        9. Weekday vs weekend Data
        10. Hotel type data analysis and Booking Channels preferred


### Steps followed 

#### Step 1 : Extract, Transform & Load

- Launch Power Query Editor, 
following operations are performed:

  (a) Promote First column as headers.

  (b) Check for Duplicate and blank Data.
  
  (c) In the given dataset Sundays and Saturdays were taken as     weekend but as per the client Fridays and saturdays are considered as weekend. 
  
  (d) Removed 'day_type' as weekend were Sun and Sat and new updated column added given in Metrics section.  
  
#### Step 2 : Data Modelling 
  
  (a) Used 'Star schema' for forming relationships

  (b) New relationships made are :
     
      Check in dates with date and Room id with Room category. 

  
  Data Model Snap:
  ![Screenshot 2024-08-05 163320](https://github.com/user-attachments/assets/c7f44fa2-d0ca-429e-aee7-745f57407c46)
 
#### Step 3 : Metrics and DAX 
  

- 1 :To get the week number from the corresponding date :


      wn = WEEKNUM(dim_date[date]) in dim_date table

 
- 2 :Based on the feedback from stakeholder, we considered 
Friday and Saturday as weekend and weekdays from Sunday to Thurdsay. In PowerBI, Sunday weekday number is 1, Monday is 2 and so on. So, if weekday number is greater than 5, then weekend or else weekday.

     day type =  Var wkd = WEEKDAY(dim_date[date],1)
     return
     IF(
     wkd>5,"Weekend","Weekday")
        
- 3 : To get the total revenue_realized 

      Revenue = SUM(fact_bookings[revenue_realized])



    ![Screenshot 2024-08-05 140908](https://github.com/user-attachments/assets/7734ddf7-8df2-4392-b035-e0ca396ab99f)


- 4 : To get the total number of bookings happened

      Total Bookings = COUNT(fact_bookings[booking_id])

 
- 5 : To get the total capacity of rooms present in hotels

      Total Capacity = SUM(fact_aggregated_bookings[capacity])

- 6 : To get the total succesful bookings happened for all hotels
      
      Succesful Bookings = SUM(fact_aggregated_bookings[successful_bookings])

- 7 : Get the average ratings given by the customers

      Average Rating = AVERAGE(fact_bookings[ratings_given])


 - 8 : To get the total number of days present in the data.
In our case, we have data from May to July. So 92 days.

    No of days = DATEDIFF(MIN(dim_date[date]),MAX(dim_date[date]),DAY) +1
 
- 9 : To get the"Cancelled" bookings out of all Total bookings happened 

      Total cancelled bookings = CALCULATE([Total Bookings],fact_bookings[booking_status]="Cancelled")
 
- 10 : calculating the cancellaton percentage.

      Cancellation % = DIVIDE([Total cancelled bookings],[Total Bookings])
 
- 11 : To get the successful 'Checked out' bookings out of all Total bookings happened

      Total Checked Out = CALCULATE([Total Bookings],fact_bookings[booking_status]="Checked Out")

- 12 :    To get the"No Show" bookings out of all Total bookings happened 

("No show" means those customers who neither cancelled nor attend to their booked rooms)

     Total no show bookings = CALCULATE([Total Bookings],fact_bookings[booking_status]="No Show")

- 13 : calculating the no show percentage.
  
      No Show rate % = DIVIDE([Total no show bookings],[Total Bookings]) 

- 14 : Occupancy means total successful bookings happened to the 
total rooms available(capacity)

     Occupancy % = DIVIDE([Total Succesful Bookings],[Total Capacity],0)
  
![Screenshot 2024-08-05 140852](https://github.com/user-attachments/assets/b40d3021-ed8f-466b-92c2-955bd529e4bb)
 - 15 : To show the percentage contribution of each booking platform for bookings in hotels.


       Booking % by Platform = DIVIDE([Total Bookings],
 CALCULATE([Total Bookings], 
 ALL(fact_bookings[booking_platform])
  ))*100
 
 - 16 : To show the percentage contribution of each room class
over total rooms booked.
       
        Booking % by Room class = DIVIDE([Total Bookings],
        CALCULATE([Total Bookings], 
        ALL(dim_rooms[room_class])
         ))*100
 
- 17 : Calculate the ADR(Average Daily rate)

It is the ratio of revenue to the total rooms booked/sold. 
It is the measure of the average paid for rooms sold in a given time period"
 
     ADR = DIVIDE( [Revenue], [Total Bookings],0)

   ![Screenshot 2024-08-05 140834](https://github.com/user-attachments/assets/f6fce99f-95f8-44f0-a1b3-845bf7e96887)


- 18 : calculate  the realisation percentage.

It is  the succesful ""checked out"" percentage over all bookings happened.
  
    Realisation % = 1- ([Cancellation %]+[No Show rate %])
    
![Screenshot 2024-08-05 140844](https://github.com/user-attachments/assets/d7873bec-5fab-4119-9b36-f0ce4a309c4c)

- 19 : Calculate the RevPAR(Revenue Per Available Room)

RevPAR represents the revenue generated per available room, whether or not they are occupied. RevPAR helps hotels measure their revenue generating performance to accurately price rooms. RevPAR can help hotels measure themselves against other properties or brands."

    RevPAR = DIVIDE([Revenue],[Total Capacity])
![Screenshot 2024-08-05 140838](https://github.com/user-attachments/assets/09e9402a-a983-46e5-956b-461692db607d)

- 20 : calculate DBRN(Daily Booked Room Nights)

This metrics tells on average how many rooms are booked for a day considering a time period

    DBRN = DIVIDE([Total Bookings], [No of days])

- 21 : calculate DSRN(Daily Sellable Room Nights)

This metrics tells on average how many rooms are ready to sell for a day considering a time period

    DSRN = DIVIDE([Total Capacity], [No of days])
    
![Screenshot 2024-08-05 140820](https://github.com/user-attachments/assets/c72cc1ce-64f2-438f-b0a6-232f8ea20613)

- 22 : calculate DURN(Daily Utilized Room Nights)

This metric tells on average how many rooms are succesfully utilized by customers for a day considering a time period
  
    DURN = DIVIDE([Total Checked Out],[No of days])

- 23 : To get the revenue change percentage week over week.

Here, 
revcw  for current week
revpw for previous week

    "Revenue WoW change % = 
    Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
    var revcw = CALCULATE([Revenue],dim_date[wn]= selv)
    var revpw =  CALCULATE([Revenue],FILTER(ALL(dim_date),dim_date[wn]= selv-1))
    return 
    DIVIDE(revcw,revpw,0)-1"


- 24 : To get the occupancy change percentage week over week.

Here, 
revcw  for current week
revpw for previous week

    "Occupancy WoW change % = 
    Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
    var revcw = CALCULATE([Occupancy %],dim_date[wn]= selv)
    var revpw =  CALCULATE([Occupancy %],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

    return

    DIVIDE(revcw,revpw,0)-1"

![Screenshot 2024-08-05 140736](https://github.com/user-attachments/assets/93da8bd6-2a92-4479-807f-3b94cb09a86b)

- 25 : To get the ADR(Average Daily rate) change percentage week over week.

Here, 
revcw  for current week
revpw for previous week

    "ADR WoW change % = 
    Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
    var revcw = CALCULATE([ADR],dim_date[wn]= selv)
    var revpw =  CALCULATE([ADR],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

    return

    DIVIDE(revcw,revpw,0)-1"
![Screenshot 2024-08-05 140750](https://github.com/user-attachments/assets/9963cdbd-96fe-430e-83c3-f94f1731ff52)

- 26 : To get the RevPar(Revenue Per Available Room) change percentage week over week.

Here, 
revcw  for current week
revpw for previous week

    "Revpar WoW change % = 
    Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
    var revcw = CALCULATE([RevPAR],dim_date[wn]= selv)
    var revpw =  CALCULATE([RevPAR],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

    return

    DIVIDE(revcw,revpw,0)-1"
    
![Screenshot 2024-08-05 140725](https://github.com/user-attachments/assets/195832b5-a9be-4074-b4b2-746047aa241f)

- 27 : To get the Realisation change percentage week over week.

Here, 
revcw  for current week
revpw for previous week

    "Realisation WoW change % = 
    Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
    var revcw = CALCULATE([Realisation %],dim_date[wn]= selv)
    var revpw =  CALCULATE([Realisation %],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

    return

    DIVIDE(revcw,revpw,0)-1" 
 ![Screenshot 2024-08-05 140801](https://github.com/user-attachments/assets/bcfad084-f8de-409a-8eb5-be7135d276d0)

- 28 : To get the DSRN(Daily Sellable Room Nights) change percentage week over week.

Here, 
revcw  for current week
revpw for previous week
 
    "DSRN WoW change % = 
    Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
    var revcw = CALCULATE([DSRN],dim_date[wn]= selv)
    var revpw =  CALCULATE([DSRN],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

    return

    DIVIDE(revcw,revpw,0)-1"

## Dashboard Designing 

- Make a matrix chart for data 
 (a) Property id 
 (b) Property name 
 (c) City 
 (d) Revenue 
 (e) Revpar
 (f) Occupancy%
 (g) ADR
 (h) DSRN
 (i) DBRN 
 (j) DURN 
 (k) Realisation%
 (l) Cancellation%
 (m) Average daily rating

 *Change revenue in million and remove the decimals for revenue and Revpar

 - Add  data bars to Revenue and average rating 
 (conditional formatting -> Data bars)

- Add slicer for 
(a) City - dropdown
(b) Room type - dropdown
(c) Month - Tile 
(d) Week no - Tile 

- Add card - Revenue
- Revenue WoW change % - card name blank then convert to table 
 go to cell elements give condition 
  
      if value > -1000 and < 0 ; Red down arrow,
      if value = 0 ; Orange neutral arrow,
      if value > 0 and < 1000 ; Green up arrow.     

- New page - Tool tip revenue
 Line chart 9(x-axis:week no; Y-axis:Revenue; Legend: Category)

select card Revenue : General :Tooltips: Report page : tooltip revenue
 Similarly, do same for RevPAR, DSRN, ADR, Occupancy%, realisation%

- Line chart:trend by key metrics
  y axis = RevPAR, ADR and Occupancy%
  x axis = Week no and Month

- Line and stacked bar chart : 
 line - ADR
 Bars - Rea;isation %
 X axis - booking platform

 switch side for ADR

- Table - day_type , RevPAR, Occupancy%, ADR and Realisation%    
 
## Conclusion

- ADR data shows cpmpany is using fixed pricing, Company should focus on Dynamic pricing like weekday-weekend pricing or holiday pricing.

- Leisure hotels are more liked so company show target that customers.

- Booking Platform shows low ADR and Realisation depicting low impact of review and ratings. the company should focus on good content online.

- Grand hotel Banglore has lowest occupancy% Manager should focus improving hotel service and components.

- During Weekend all levers are high should increase prices.

- Average rating shows company needs to work on Customer satisfaction.

- Company can start its own booking site to reduce commision.

- Use of Promotion coupons and Complimentary treats to promote sales.


## Dashboard 

![Screenshot 2024-08-05 140549](https://github.com/user-attachments/assets/f29c5220-3adb-44ed-946e-6bc158ace6c0)
