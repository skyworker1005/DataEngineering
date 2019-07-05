# BDE_4_YelpDataAnalysis.md
    20190705-DataEngineering (Con't)
    Title: Yelp Restaurants Big Data Analysis Project Tutorial
        
        
## Yelp Restaurants Big Data Analysis Project Tutorial
 
 
### [Intro & Objectives]
    Dataset: 2017 Yelp challange data set 
        1 TAR file - 2.28 GB compressed
        6 JSON files - 5.79 GB uncompressed
        (The included files are: business, reviews, user, check-in, tip and photos)
        
    Objectives
        Download and extract the data set from Yelp
        Upload JSON files to HDFS using Ambari
        Install Rcongui SerDe
        Create tables and queries using HiveQL
        Export results and visualize them in Tableau


### [Step 1: Data Preparation]
    Source Link: http://bit.ly/SKT_COMB_LAB
    # tar -zxvf yelp_dataset.tar
        /dataset/business.json, checkin.json, photos.json, review.json, tip.json, user.json
    
### [Step 2: Uploading Data to HDFS For Storage and Analysis]
    hdfs dfs -mkdir /user/training/yelp
    hdfs dfs -mkdir /user/training/yelp/business 
    hdfs dfs -mkdir /user/training/yelp/review 
    hdfs dfs -mkdir /user/training/yelp/users 
    hdfs dfs -mkdir /user/training/yelp/tip 
    hdfs dfs -mkdir /user/training/yelp/checkin 

    cd /home/training/dataset
    hdfs dfs -put business.json /user/training/yelp/business/
    hdfs dfs -put checkin.json /user/training/yelp/checkin/
    hdfs dfs -put review.json /user/training/yelp/review/
    hdfs dfs -put tip.json /user/training/yelp/tip/
    hdfs dfs -put user.json /user/training/yelp/users/
    #hdfs dfs -put photos.json /user/training/yelp/photos/
    
### [Step 3: Adding RCONGUI JSON SerDe]
    (this step may not be necessary with current hive version) 

    wget -O json-serde-1.3.8-jar-with-dependencies.jar  http://www.congiu.net/hive-json-serde/1.3.8/cdh5/json-serde-1.3.8-jar-with-dependencies.jar

    wget -O json-udf-1.3.8-jar-with-dependencies.jar http://www.congiu.net/hive-json-serde/1.3.8/cdh5/json-udf-1.3.8-jar-with-dependencies.jar

    ADD JAR json-serde-1.3.8-jar-with-dependencies.jar; 
    ADD JAR json-udf-1.3.8-jar-with-dependencies.jar; 

    set hive.cli.print.header=true;


### [Step 4: Creating Tables in HIVE]

    * 테이블 생성쿼리

        CREATE EXTERNAL TABLE business4 (
        address string,
        business_id string,
        categories array<string>,
        city string,
        hours struct<friday:string, monday:string, saturday:string, sunday:string, thursday:string,
        tuesday:string, wednesday:string>,
        is_open int,
        latitude double, 
        longitude double,
        name string,
        neighborhood string,
        postal_code string,
        review_count int,
        stars double,
        state string,
        Attributes struct<
        Accepts_Insurance:boolean,
        Ages_Allowed:string,
        Alcohol:string,
        Bike_Parking:boolean,
        Business_Accepts_Bitcoin:boolean,
        Business_Accepts_Credit_Cards:boolean,
        By_Appointment_Only:boolean,
        Byob:boolean,
        BYOB_Corkage:string,
        Caters:boolean,
        Coat_Check:boolean,
        Corkage:boolean,
        Dogs_Allowed:boolean, 
        Drive_Thru:boolean,
        Good_For_Dancing:boolean,
        Good_For_Kids:boolean,
        Happy_Hour:boolean,
        Has_TV:boolean,
        Noise_Level:string,
        Open24Hours:boolean,
        Outdoor_Seating:boolean,
        Restaurants_Attire:string,
        Restaurants_Counter_Service:boolean, 
        Restaurants_Delivery:boolean,
        Restaurants_Good_For_Groups:boolean,
        Restaurants_Reservations:boolean,
        Restaurants_Table_Service:boolean,
        Restaurants_Take_Out:boolean,
        Smoking:string,
        WheelchairAccessible:boolean, 
        WiFi:string,
        Ambience:struct<
        Casual:boolean,
        Classy:boolean,
        Divey:boolean,
        Hipster:boolean,
        Intimate:boolean,
        Romantic:boolean,
        Touristy:boolean,
        Trendy:boolean,
        Upscale:boolean>,
        BestNights:struct<
        Friday1:boolean,
        Monday1:boolean,
        Saturday1:boolean, 
        Sunday1:boolean,
        Thursday1:boolean,
        Tuesday1:boolean,
        Wednesday1:boolean>,
        BusinessParking:struct<
        Garage:boolean,
        Lot:boolean,
        Street:boolean,
        Valet:boolean,
        Validated:boolean>,
        DietaryRestrictions:struct<
        Dairy_Free:boolean,
        Gluten_Free:boolean,
        Halal:boolean,
        Kosher:boolean,
        Soy_Free:boolean,
        Vegan:boolean,
        Vegetarian:boolean>,
        GoodForMeal:struct<
        Breakfast:boolean,
        Brunch:boolean,
        Dessert:boolean, 
        Dinner:boolean,
        Latenight:boolean,
        Lunch:boolean>,
        HairSpecializesIn:struct<
        Africanamerican:boolean,
        Asian:boolean,
        Coloring:boolean,
        Curly:boolean,
        Extensions:boolean,
        Kids:boolean,
        Perms:boolean,
        Straightperms:boolean>,
        Music:struct<
        BackgroundMusic:boolean,
        Dj:boolean,
        Jukebox:boolean,
        Karaoke:boolean,
        Live:boolean, 
        NoMusic:boolean,
        Video:boolean>,
        restaurantspricerange2:int>)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/business'; 


        CREATE TABLE restaurants
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/business/restaurants'
        AS
        SELECT * FROM exploded WHERE cat_exploded="Restaurants"; 

        CREATE EXTERNAL TABLE review (
        business_id string,
        cool int,
        review_date string,
        funny int,
        review_id string,
        stars int,
        text string,
        useful int,
        user_id string)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/review'; 

        CREATE TABLE review_filtered
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE 
        LOCATION '/user/training/yelp/review_filtered'
        AS
        SELECT re.business_id, r.stars, r.user_id
        FROM review r JOIN restaurants re
        ON r.business_id = re.business_id;

        CREATE EXTERNAL TABLE users (
        average_stars double,
        compliment_cool int,
        compliment_cute int,
        compliment_funny int,
        compliment_hot int,
        compliment_list int,
        compliment_more int,
        compliment_note int,
        compliment_photos int,
        compliment_plain int,
        compliment_profile int,
        compliment_writer int,
        cool int,
        elite array<int>,
        fans int,
        friends array<string>,
        funny int,
        name string,
        review_count int,
        useful int,
        user_id string,
        yelping_since string)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        LOCATION '/user/training/yelp/users'; 

        CREATE TABLE elite_users
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/users/elite'
        AS
        SELECT * FROM users LATERAL VIEW explode(elite) c AS elite_year; 

        CREATE EXTERNAL TABLE tip (
        text string,
        date_tip string,
        likes int,
        business_id string,
        user_id string)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/tip'; 
        
### [Analysis]

#### 1. bussiness json파일을 hive schema 없이 테이블로 넣기

##### 1) spark >>  json 스트링에서 parquet에서 사용불가한 문자인 '-'를 '_'으로 변환하여 pqrquet 파일로 쓰기
    val rdd = spark.read.textFile("hdfs:/user/training/yelp/business").map(value => value.replace("dairy-free", "dairy_free").replace("gluten-free","gluten_free").replace("soy-free","soy_free")).rdd
    val df = spark.read.json(rdd)
    df.write.format("parquet").save("hdfs:/user/training/max/business_parquet")

##### 2) hdfs 에서 file 경로 복사
    
##### 3) impala에서 테이블 처리 : impala에서는 parquet 파일을 기반으로 impala table을 생성할 수 있음
    CREATE EXTERNAL TABLE test.business_max LIKE PARQUET 'hdfs:/user/training/max/business_parquet/part-00000-5928b258-0027-48b6-bc5a-9296f7d2690a-c000.snappy.parquet'
      STORED AS PARQUET
      LOCATION 'hdfs:/user/training/max/business_parquet';
 
#### 2. Yelp data 분석하기

##### * 참고용 정보 : business table schema
    address                     string                         from deserializer
    business_id              string                         from deserializer
    categories                array<string>          from deserializer
    city                             string                         from deserializer
    hours                         struct<friday:string,monday:string,saturday:string,sunday:string,thursday:string,tuesday:string,wednesday:string>                 from deserializer
    is_open                     int                              from deserializer
    latitude                     double                       from deserializer
    longitude                  double                       from deserializer
    name                         string                         from deserializer
    neighborhood        string                         from deserializer
    postal_code             string                         from deserializer
    review_count          int                              from deserializer
    stars                          double                       from deserializer
    state                          string                         from deserializer
    attributes                            struct<accepts_insurance:boolean,ages_allowed:string,alcohol:string,bike_parking:boolean,business_accepts_bitcoin:boolean,business_accepts_credit_cards:boolean,by_appointment_only:boolean,byob:boolean,byob_corkage:string,caters:boolean,coat_check:boolean,corkage:boolean,dogs_allowed:boolean,drive_thru:boolean,good_for_dancing:boolean,good_for_kids:boolean,happy_hour:boolean,has_tv:boolean,noise_level:string,open24hours:boolean,outdoor_seating:boolean,restaurants_attire:string,restaurants_counter_service:boolean,restaurants_delivery:boolean,restaurants_good_for_groups:boolean,restaurants_reservations:boolean,restaurants_table_service:boolean,restaurants_take_out:boolean,smoking:string,wheelchairaccessible:boolean,wifi:string,ambience:struct<casual:boolean,classy:boolean,divey:boolean,hipster:boolean,intimate:boolean,romantic:boolean,touristy:boolean,trendy:boolean,upscale:boolean>,bestnights:struct<friday1:boolean,monday1:boolean,saturday1:boolean,sunday1:boolean,thursday1:boolean,tuesday1:boolean,wednesday1:boolean>,businessparking:struct<garage:boolean,lot:boolean,street:boolean,valet:boolean,validated:boolean>,dietaryrestrictions:struct<dairy_free:boolean,gluten_free:boolean,halal:boolean,kosher:boolean,soy_free:boolean,vegan:boolean,vegetarian:boolean>,goodformeal:struct<breakfast:boolean,brunch:boolean,dessert:boolean,dinner:boolean,latenight:boolean,lunch:boolean>,hairspecializesin:struct<africanamerican:boolean,asian:boolean,coloring:boolean,curly:boolean,extensions:boolean,kids:boolean,perms:boolean,straightperms:boolean>,music:struct<backgroundmusic:boolean,dj:boolean,jukebox:boolean,karaoke:boolean,live:boolean,nomusic:boolean,video:boolean>,restaurantspricerange2:int>         from deserializer
    cat_exploded          string                         from deserializer

##### 1) 공통적으로 사용할 view 생성
    cateogries 컬럼을 explode해서 view 로 생성

    CREATE VIEW r_view AS
    SELECT Restaurants.*, category
    FROM Restaurants LATERAL VIEW explode(categories) a AS category

##### 2) Which cities have the highest number of restaurants?
    >> query
        select city, count(1) as cnt from Restaurants group by city  order by cnt desc limit 1;
    >> result
        Toronto                7148

##### 3) Top 15 sub-categories of restaurants
    >> query
        SELECT category, count(1) as cnt
        FROM Restaurants LATERAL VIEW explode(categories) a AS category
        GROUP BY category
        order by cnt desc
        limit 15
        ;

         SELECT category, count(1) as cnt
        FROM r_view
        GROUP BY category
        order by cnt desc
        limit 15
        ;
    >> result
        Restaurants         54618
        Food     10348
        Nightlife               7479
        Bars     7194
        Sandwiches          6345
        Fast Food              6280
        American (Traditional)       6097
        Pizza    6067
        Italian  4662
        Burgers                 4558
        Breakfast & Brunch            4497
        Mexican               4105
        Chinese                 3987
        American (New) 3979
        Cafes    3039

##### 4) Distribution of ratings vs. categories
    >> query
        SELECT category, stars, count(1) as cnt
        FROM r_view
        GROUP BY category, stars
        order by cnt desc
        limit 20
        ;
    >> result
        Restaurants         4.0            13526
        Restaurants         3.5            13387
        Restaurants         3.0            9825
        Restaurants         4.5            6516
        Restaurants         2.5            5448
        Restaurants         2.0            2945
        Food     4.0            2919
        Food     3.5            2179
        Nightlife               3.5            2133
        Food     4.5            2079
        Nightlife               4.0            2057
        Bars     3.5            2052
        Bars     4.0            1990
        American (Traditional)       3.5            1631
        Sandwiches          4.0            1566
        Restaurants         5.0            1492
        Nightlife               3.0            1486
        American (Traditional)       3.0            1461
        Pizza    3.5            1454
        Pizza    4.0            1450

##### 5) What ratings do the majority of restaurants have?
    >> query
        SELECT stars, count(business_id) as cnt
            FROM r_view
            GROUP BY stars
            order by cnt desc
            limit 20
            ;
    >> result
        4.0        54675
        3.5        51371
        3.0        36258
        4.5        27814
        2.5        19421
        2.0        9932
        5.0        6770
        1.5        3612
        1.0        1103

##### 6) Rating distribution in restaurant reviews
    select stars, round((count(stars) / sum(count(stars)) over())* 100)
    from review
     group by stars
    ;
    5               43.0
    4               23.0
    3               12.0
    2               8.0
    1               14.0


(end of file) 
