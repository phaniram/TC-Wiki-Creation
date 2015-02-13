HP Haven Twitter Analysis Tutorial Challenge
================

**[Scenario]** 

Let's say your company launched a product and wanted to know it's views in public or company has undergone some share market changes and to see what people views are on your stock symbol? How do you go about it?

**[Behaviour]** 

_SurveyMonkey?_

People won't respond to surveys and also don't take much interest in taking it unless you bribe them with gifts or sweepstakes.

What if we can analyse public reviews available in twitter or facebook comments?

Well, here's the tutorial to help you do that.

In this tutorial we'll be creating a web application, which does sentiment analysis on public data i.e tweets through IDOLOnDemand SentimentAnalysis API and store the information in Vertica Instance, display it in nice Graph over the timeline.

**[Impact]**

You'll now know exactly what people are thinking about your product/stock symbol over the timeline of tweets about the product you launched/recent doings in stock market. 

Nice isn't it? Then let's start.


## Prerequisites
1. Vertica VM  
   For Vertica Setup in local VMWare instance follow 
   http://www.topcoder.com/challenge-details/30048444/?type=develop&noncache=true
2. IDOL OnDemand API Key	
   Register at : http://www.idolondemand.com/signup.html
   Obtain API Key at : https://www.idolondemand.com/developer/apis
3. Twitter Keys and Access Tokens  
   * Sign-In to twitter and navigate to https://apps.twitter.com/app/new to create a new app
   * Fill in the details and create twitter application
   * Navigate to app created above. Generate Access Token and Token Secret.
   * You'll now have following Keys/Tokens	
      Consumer Key (API Key)	
      Consumer Secret (API Secret)	
      Access Token	
      Access Token Secret	
4. Create a Database "**SOCIAL_SENTIMENT_REPO**" Vertica.
5. Create table "**TWEET_SENTIMENT**" on "**SOCIAL_SENTIMENT_REPO**" with following script


        CREATE TABLE IF NOT EXISTS TWEET_SENTIMENT (
		ID INTEGER NOT NULL PRIMARY KEY,
		CREATED_AT TIMESTAMP NOT NULL,
		TEXT VARCHAR(300),
		SEARCH_TEXT VARCHAR(25),
		SCREEN_NAME VARCHAR(50) NOT NULL,
		AGG_SENTIMENT VARCHAR(15),
		AGG_SCORE VARCHAR(30)
        )

## TweetSentiment App

Create a new "Dynamic Web Project" in Eclipse   

  Add following files to the project.

_To process tweets or comments and to maintain history of our analysis on them, we should be storing them. So, we'll use Vertica DB. This Class helps us to get a DB Connection to run our queries on VerticaDB._

- Create "**VerticaDBUtil.java**"   
  Edit **DB_CONNECTION** according to your **VM_IP** and **VM_VERTICA_DB_NAME**.  
    

        import java.sql.Connection;
        import java.sql.DriverManager;
        import java.sql.ResultSet;
        import java.sql.SQLException;
        import java.sql.Statement;

        /*
        * HelperClass for making DBConnections to VerticaDB
        */
        public class VerticaDBUtil {

        private static Connection conn;
        private static final String DB_DRIVER = "com.vertica.jdbc.Driver";
        private static final String DB_CONNECTION = "jdbc:vertica://<VM_IP>:5433/VM_VERTICA_DB_NAME";
        private static final String DB_USER = "dbadmin";
        private static final String DB_PASSWORD = "password";

        //Check for required DB_DRIVER class existence
        static {
            try {
                Class.forName(DB_DRIVER);
            } catch (ClassNotFoundException ex) {
                ex.printStackTrace();
            }
        }

        //Get DB Connection
        public static Connection getDBConnection() throws SQLException {
            return conn = DriverManager.getConnection(DB_CONNECTION, DB_USER,
                    DB_PASSWORD);
        }

        
        //Begin Transaction, set's AutoCommit to false.
        public static void beginTransaction() {
            if (conn != null) {
                try {
                    conn.setAutoCommit(false);
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
        }

        //Commit in Connection
        public static void commit() {
            if (conn != null) {
                try {
                    conn.commit();
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
        }

        //Rollback in Connection
        public static void rollback() {
            if (conn != null) {
                try {
                    conn.rollback();
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
        }

        //Close all resources
        public static void closeDBUtil(ResultSet rs, Statement stmt, Connection conn) {
            try {
                if (rs != null) {
                    rs.close();
                    rs = null;
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }

            try {
                if (stmt != null) {
                    stmt.close();
                    stmt = null;
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }

            try {
                if (conn != null) {
                    conn.close();
                    conn = null;
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        }
        }


_Each tweet we retrieve and analyse , we'll be storing them each as a record in a table, this is our perisistant storage, which can be used to retrieve and plot. This class helps us to insert our analyzed data along with supporting information into VerticaDB._


- Create "**TweetSentimentTabUtil.java**"   


        import java.sql.Connection;
        import java.sql.PreparedStatement;
        import java.sql.ResultSet;
        import java.sql.SQLException;
        import java.sql.Statement;

        import com.cypronmaya.jdbc.VerticaDBUtil;

        import twitter4j.Status;

        //DAO for TweetSentiment Table in Vertica DB

        public class TweetSentimentTabUtil {

		//Get latest TweetID we pushed into DB
		public long getLatestTweetSentimentEntryID(String searchText) throws Exception{
	        Connection conn = null;
	        Statement stmt = null;
	        long latestId = 0;
			String sql = "SELECT NVL(MAX(ID),-1) MAX_ID FROM TWEET_SENTIMENT WHERE SEARCH_TEXT = "+searchText;
	        try{
	        conn = VerticaDBUtil.getDBConnection();
	        stmt = conn.createStatement();
	        VerticaDBUtil.beginTransaction(); // Begin Transaction
	        ResultSet rs = stmt.executeQuery(sql); // Execute Statement
	        if (rs.next()) {
	        	latestId = rs.getLong("MAX_ID");
	        	VerticaDBUtil.commit(); // Commit Transaction
	        } else {
	        	VerticaDBUtil.rollback(); // Roleback Transaction
	        }
	        } finally {
	        	VerticaDBUtil.closeDBUtil(null, stmt, conn);  // Cleanup Resources	
			}
			return latestId; 	                
		}

		//Get SentimentData for a searchText stored in DB
		public JSONArray getSentimentData(String searchText) throws Exception{
	        Connection conn = null;
	        PreparedStatement pstmt = null;
	        ResultSet rs = null;
	        JSONArray tweetSentiments = new JSONArray();
			try{
			conn = VerticaDBUtil.getDBConnection();
			//Fetch all the sentiment_data stored in vertica DB for analysis.
			//SentimentData needs to be sorted by DateTime to be properly plotted in HighCharts.
	        pstmt = conn.prepareStatement("SELECT * FROM TWEET_SENTIMENT WHERE SEARCH_TEXT = ? ORDER BY CREATED_AT ASC");
	        pstmt.setString(1, searchText);
	        rs = pstmt.executeQuery();
	        //Get ResultSet formed as JSON response for use in HighCharts	        
	        while (rs.next()) {
	        	JSONObject tweetSentiment = new JSONObject();
	        	tweetSentiment.put("SCREEN_NAME", rs.getString("SCREEN_NAME"));
	        	tweetSentiment.put("CREATED_AT", rs.getTimestamp("CREATED_AT").getTime());
	        	tweetSentiment.put("AGG_SENTIMENT", rs.getString("AGG_SENTIMENT"));
	        	tweetSentiment.put("AGG_SCORE", Double.parseDouble(rs.getString("AGG_SCORE")));
	        	tweetSentiment.put("TEXT", rs.getString("TEXT"));
	        	tweetSentiments.add(tweetSentiment);
	        }
			} finally {
	        	VerticaDBUtil.closeDBUtil(rs, pstmt, conn);  // Cleanup Resources	
			}
			return tweetSentiments;
		}

        //Creates a Tweet Sentiment record
        public int createTweetSentimentEntry(Status tweet, String searchText, String sentiment, String score) throws Exception {
	        Connection conn = null;
	        PreparedStatement pstmt = null;
	        try{
	        conn = VerticaDBUtil.getDBConnection();
	        String sql = "INSERT INTO TWEET_SENTIMENT(ID,CREATED_AT,TEXT,SEARCH_TEXT,SCREEN_NAME,AGG_SENTIMENT,AGG_SCORE) VALUES(?, ?, ?, ?, ?, ?, ?)";	        
	        pstmt = conn.prepareStatement(sql);
	        
	        pstmt.setLong(1, tweet.getId());
	        pstmt.setTimestamp(2, new java.sql.Timestamp(tweet.getCreatedAt().getTime()));
	        pstmt.setString(3, tweet.getText());
	        pstmt.setString(4, searchText);
	        pstmt.setString(5, tweet.getUser().getScreenName());
	        pstmt.setString(6, sentiment);
	        pstmt.setString(7, score);
	        
	        VerticaDBUtil.beginTransaction(); // Begin Transaction
	        int result = pstmt.executeUpdate(); // Execute Statement
	        if (result != 0) {
	        	VerticaDBUtil.commit(); // Commit Transaction
	        } else {
	        	VerticaDBUtil.rollback(); // Roleback Transaction
	        }
	        } finally {
	        	VerticaDBUtil.closeDBUtil(null, pstmt, conn);  // Cleanup Resources	
			} 	        

            }
            }


_Now we need to analyse sentiment on the tweets we retrieve, to do that we will be passing tweet data to IDOLOnDemand SentimentAnalysis API. This class helps us to pass tweet data to IDOLOnDemand SentimentAnalysis API and get sentiment aggregate,score for our use. More the positive, more the chance that our product/mark is having a good response in public._


- Create "**IdolOnDemand.java**"    
  Replace API_KEY with IdolOnDemand API Key


        import java.io.BufferedReader;
        import java.io.IOException;
        import java.io.InputStream;
        import java.io.InputStreamReader;
        import java.io.Reader;
        import java.net.URL;
        import java.net.URLEncoder;
        import java.nio.charset.Charset;
        import java.util.HashMap;

        import org.json.simple.JSONObject;
        import org.json.simple.JSONValue;

        public class IdolOnDemand {

        private static String apikey = <API_KEY>;

        //Read to String
        private static String readToString(Reader rd) throws IOException {
            StringBuilder sb = new StringBuilder();
            char[] data = new char[1024];
            int xf = 0;
            while((xf = rd.read(data, 0, 1024)) > 0) {
                sb.append(data, 0, xf);
            }
            return sb.toString();
        }
        
        //Connect to API Endpoint along with parameters passed in. 
        private static String callApi(String query) {
            try {
                System.out.println(query);
                URL url = new URL(query);
                InputStream is = url.openStream();
                BufferedReader br = new BufferedReader(
                    new InputStreamReader(is, Charset.forName("UTF-8")));
                String output = readToString(br);
                return output;
            } catch(Exception ex) {
                ex.printStackTrace();
                return null;
            }
        }
    
	    
	    //Identifies language of given text
	    public HashMap<String, String> identifyLanguage(String text) {
	        try {
	        	HashMap<String,String> hmap = new HashMap<String,String>();
	            String query = "https://api.idolondemand.com/1/api/sync/identifylanguage/v1" +
	                    "?text=" + URLEncoder.encode(text, "UTF-8") +
	                    "&apikey=" + apikey;
	            
	            String output = callApi(query);
	            
	            JSONObject obj = (JSONObject)JSONValue.parse(output);
	            String language = (String)((JSONObject)obj).get("language");
	            String language_iso639_2b = (String)((JSONObject)obj).get("language_iso639_2b");
	            hmap.put("language", language);
	            hmap.put("language_iso639_2b", language_iso639_2b);
	            System.out.println(text + ", value: " + language);            
	            return hmap;
	        } catch(Exception ex) {
	            ex.printStackTrace();
	            return null;
	        }
	    }
		
	
        
        //Runs sentiment analysis on given text
        public HashMap<String, String> executeSentimentAnalysis(String text) {
            try {
            	HashMap<String,String> hmap = new HashMap<String,String>();
                String query = "https://api.idolondemand.com/1/api/sync/analyzesentiment/v1" +
                        "?text=" + URLEncoder.encode(text, "UTF-8") +
                        "&apikey=" + apikey;
                
                String output = callApi(query);
                
                JSONObject obj = (JSONObject)JSONValue.parse(output);
                JSONObject aggregate = (JSONObject)obj.get("aggregate");
                String sentiment = (String)aggregate.get("sentiment");
                Object score = aggregate.get("score");
                hmap.put("sentiment", sentiment);
                hmap.put("score", score.toString());
                System.out.println(sentiment + ", value: " + score);            
                return hmap;
            } catch(Exception ex) {
                ex.printStackTrace();
                return null;
            }
        }
        }

_Our one source of reviews is tweets. To get them we'll be searching twitter on some unique tag or hashkey which represents our newly launched product or stock symbol. Tweets retrieved here are passed to sentiment analysis, we'll be then storing the response and tweet information in VerticaDB._


- Create "**App.java**". Replace

  CONSUMER_KEY -> Consumer Key (API Key)	
  CONSUMER_SECRET -> Consumer Secret (API Secret)	 
  ACCESS_TOKEN -> Access Token	
  ACCESS_TOKEN_SECRET -> Access Token Secret  


        import java.util.ArrayList;
        import java.util.HashMap;
        import java.util.List;

        import twitter4j.Query;
        import twitter4j.QueryResult;
        import twitter4j.Status;
        import twitter4j.Twitter;
        import twitter4j.TwitterException;
        import twitter4j.TwitterFactory;
        import twitter4j.auth.AccessToken;

        //Main App that fetches tweets and run sentiment analysis , store them to Vertica DB
        public class App {

        public static void main(String[] args) {        
            String searchText = "$HPQ";
            Twitter twitter = TwitterFactory.getSingleton();
			TweetSentimentTabUtil tweetDBUtil = new TweetSentimentTabUtil();

            //Set Twitter App Keys to authenticate and fetch tweets.
            AccessToken accessToken = new AccessToken(<ACCESS_TOKEN>, <ACCESS_TOKEN_SECRET>);
            twitter.setOAuthConsumer(<CONSUMER_KEY>, <CONSUMER_SECRET>);
            twitter.setOAuthAccessToken(accessToken);

            try {
            Query query = new Query(searchText);
	 	    try {
				long latestId = tweetDBUtil.getLatestTweetSentimentEntryID(searchText);
				if(latestId!=-1){
					query.setSinceId(latestId);
				}
			} catch (Exception e1) {
				e1.printStackTrace();
			}
            QueryResult result;
            List<Status> totalTweets = new ArrayList<Status>();
                do {
                    result = twitter.search(query);
                    List<Status> tweets = result.getTweets();
                    //Accumulate tweets
                    totalTweets.addAll(tweets);
                    for (Status tweet : tweets) {
                        System.out.println("@" + tweet.getUser().getScreenName() + " - " + tweet.getText());
                    }
                } while ((query = result.nextQuery()) != null);
                
                IdolOnDemand idolOnDemand = new IdolOnDemand(); 
                //Run sentiment Analysis on each tweet and store it as a record into vertica DB.
                for(Status tweet : totalTweets)
                {
                    try {
					HashMap<String,String> langResp = idolOnDemand.identifyLanguage(tweet.getText());
					String tweetLang = langResp.get("language_iso639_2b");
					//If language is unidentified, treat it as english. ref: https://www.idolondemand.com/developer/apis/identifylanguage#response
		        	if(tweetLang.equalsIgnoreCase("UND")) tweetLang = "ENG"; 
		        	HashMap<String,String> resp = idolOnDemand.executeSentimentAnalysis(tweet.getText(),tweetLang);				
		        	tweetDBUtil.createTweetSentimentEntry(tweet, searchText,resp.get("sentiment"), resp.get("score"));			
                    } catch (Exception e) {
                        e.printStackTrace();
                    }           
                }
                System.exit(0);
            } catch (TwitterException te) {
                te.printStackTrace();
                System.out.println("Failed to search tweets: " + te.getMessage());
                System.exit(-1);
            }
        }
        }


_Now we're done with tweet retrievals, sentiment analysis, storing. This Servlet helps us to retrieve the stored data from VerticaDB based on the search Text, to be used in Plotting Graph later._


- Create a new servlet "**SentimentDataRetriever.java**"  


		import java.io.IOException;
		import java.io.PrintWriter;
		import javax.servlet.ServletException;
		import javax.servlet.annotation.WebServlet;
		import javax.servlet.http.HttpServlet;
		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletResponse;
		
		import org.json.simple.JSONArray;

        //Servlet to retrieve SentimentData on given text.
        @WebServlet("/SentimentDataRetriever")
        public class SentimentDataRetriever extends HttpServlet {
        private static final long serialVersionUID = 1L;

		private static TweetSentimentTabUtil tweetDBUtil;
	    public SentimentDataRetriever() {
	    	tweetDBUtil = new TweetSentimentTabUtil();
	    }
	
		@SuppressWarnings("unchecked")
		protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
			//Fetch searchText Parameter
			String searchText = request.getParameter("searchText");
			JSONArray tweetSentiments = new JSONArray();
			try {
				tweetSentiments = tweetDBUtil.getSentimentData(searchText);
			} catch (Exception e) {
				e.printStackTrace();
			}
			response.setContentType("application/json");
		    PrintWriter out = response.getWriter();
		    out.print(tweetSentiments);
		    out.flush();		
		}

        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            doGet(request,response);
        }
        }

_Everything at hand, we need nice representation of our sentiments on tweets over the timeline. I chose highcharts here (you're free to use any), so in this html we'll be making an ajax request to above servlet with a parameter and get data, and plot it in the timeline graph._


- Create "**index.html**" in WebContent folder


        <!DOCTYPE html>
        <html>
        <head>
        <meta charset="UTF-8">
        <title>IdolOnDemand Sentiment Analysis</title>
        <script src="https://code.jquery.com/jquery-2.1.3.js"></script>
        <script src="http://code.highcharts.com/highcharts.js"></script>
        <script src="http://code.highcharts.com/highcharts-more.js"></script>
        </head>
        <body>
        <div id="container" style="min-width: 310px; height: 400px; margin: 0 auto"></div>

        <script type="text/javascript">

        //Loads sentiment Data with given searchText
        loadSentimentData("$HPQ");

        function loadSentimentData(searchTxt){
        //Makes GET request to Servlet
        $.getJSON( "SentimentDataRetriever", { searchText: searchTxt})
        .done(function( data ) {
            var tweetSentimentData = [];
              $.each( data, function( i, tweetSentiment ) {
                          tweetSentimentData.push([tweetSentiment.CREATED_AT,tweetSentiment.AGG_SCORE]);
              });
            $(function () {
                $('#container').highcharts({
                    chart: {
                        type: 'spline',
                        zoomType: 'x'
                    },
                    title: {
                        text: 'IdolOnDemand Sentiment Analysis'
                    },
                    subtitle: {
                        text: "Search Text : " + searchTxt
                    },
                    xAxis: {
                        type: 'datetime',
                        dateTimeLabelFormats: { 
                            minute: '%H:%M',
                            hour: '%H:%M',
                            day: '%e. %b',
                            week: '%e. %b',
                            month: '%b \'%y',
                            year: '%Y'
                        },
                        title: {
                            text: 'DateTime'
                        }
                    },
                    yAxis: {
                        title: {
                            text: 'Sentiment Score'
                        }        },
                    tooltip: {
                        headerFormat: '<b>{series.name}</b><br>',
                        pointFormat: '{point.x:%e %b %H:%M}: {point.y:.4f}'
                    },
                    credts: { enabled: false},
                    legend: { enabled: false},          
                    plotOptions: {
                        spline: {
                            marker: {
                                enabled: true
                            }
                        }
                    },
                    series: [{
                        name: 'TweetSentiments',
                        negativeColor: '#0F0F0F',
                        data: tweetSentimentData
                    }]
                });
            });
        });
        };
        </script>
        </body>
        </html>

Running the application upon opening "index.html" would produce a graph on Tweets Sentiment over the timeline.

![tweetsentimentgraph](https://cloud.githubusercontent.com/assets/231750/5991252/a912049e-aa07-11e4-907e-2b865b0d8c03.png)

Modify index.html to following 


			  tweetSentimentData.push({
				  	x: tweetSentiment.CREATED_AT,
				  	y: tweetSentiment.AGG_SCORE,
					name: tweetSentiment.SCREEN_NAME,
					tweet: tweetSentiment.TEXT,
					sentiment: tweetSentiment.AGG_SENTIMENT
				  	});
-----------------------------------------------                    
		        tooltip: {
		            headerFormat: '<b>@{point.key}</b> {point.x:%e %b %H:%M}<br>',
		            pointFormat: '{point.tweet}<br/>Score : <b>{point.y:.4f}</b>'
		        },

You can now even see the tweet at that point, and screen name.

![tweetdatasentimentgraph](https://cloud.githubusercontent.com/assets/231750/5993354/ead2e5a8-aa73-11e4-9a85-30c1405c55f0.png)

_Now you'll see during the period where your stock symbol making good impression in public with consecutive or max positives._