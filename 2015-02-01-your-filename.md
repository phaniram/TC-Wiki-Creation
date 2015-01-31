HP Haven Twitter Analysis Tutorial Challenge
================

This is a guide to teach you how to use IdolOnDemand SentimentAnalysis API on Tweets. 

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

## Sample App

* Create a new "Dynamic Web Project" in Eclipse   

- Create "VerticaDBUtil.java"   
  Edit DB_CONNECTION according to your _VMIP_ and _VMVERTICADBNAME_


	import java.sql.Connection;
	import java.sql.DriverManager;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;

	public class VerticaDBUtil {

		private static Connection conn;
		private static final String DB_DRIVER = "com.vertica.jdbc.Driver";
		private static final String DB_CONNECTION = "jdbc:vertica://<VM_IP>:5433/<VM_VERTICA_DB_NAME>";
		private static final String DB_USER = "dbadmin";
		private static final String DB_PASSWORD = "password";

		static {
			try {
				Class.forName(DB_DRIVER);
			} catch (ClassNotFoundException ex) {
				ex.printStackTrace();
			}
		}

		public static Connection getDBConnection() throws SQLException {
			return conn = DriverManager.getConnection(DB_CONNECTION, DB_USER,
					DB_PASSWORD);
		}

		public static void beginTransaction() {
			if (conn != null) {
				try {
					conn.setAutoCommit(false);
				} catch (SQLException ex) {
					ex.printStackTrace();
				}
			}
		}

		public static void commit() {
			if (conn != null) {
				try {
					conn.commit();
				} catch (SQLException ex) {
					ex.printStackTrace();
				}
			}
		}

		public static void rollback() {
			if (conn != null) {
				try {
					conn.rollback();
				} catch (SQLException ex) {
					ex.printStackTrace();
				}
			}
		}

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

- Create "TweetsUtil.java"   



	import java.util.HashMap;
	import java.util.List;
	import twitter4j.Query;
	import twitter4j.QueryResult;
	import twitter4j.Status;
	import twitter4j.Twitter;
	import twitter4j.TwitterException;
	import twitter4j.TwitterFactory;

	public class TweetsUtil {

		public void getTweetsByTag(String tagName){
		    Twitter twitter = TwitterFactory.getSingleton();
		    Query query = new Query(tagName);
		    QueryResult result;
			try {
				result = twitter.search(query);
			    for (Status status : result.getTweets()) {		    	
			        System.out.println("@" + status.getUser().getScreenName() + ":" + status.getText());
			    }			
			} catch (TwitterException e) {
				e.printStackTrace();
			}
		}

		public void pushTweetsToDB(List<Status> tweets,String searchText){
			TweetSentimentTabUtil tweetDBUtil = new TweetSentimentTabUtil();
			IdolOnDemand idolOnDemand = new IdolOnDemand();		
			for(Status tweet : tweets)
			{
				try {
					HashMap<String,String> resp = idolOnDemand.executeSentimentAnalysis(tweet.getText());				
					tweetDBUtil.createTweetSentimentEntry(tweet, searchText,resp.get("sentiment"), resp.get("score"));			
				} catch (Exception e) {
					e.printStackTrace();
				}			
			}
		}
	}

- Create "TweetSentimentTabUtil.java"   


	import java.sql.Connection;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;

	import com.cypronmaya.jdbc.VerticaDBUtil;

	import twitter4j.Status;

	public class TweetSentimentTabUtil {
		
		public int createTweetSentimentEntry(Status tweet, String searchText, String sentiment, String score) throws Exception {
		        String sql = "INSERT INTO TWEET_SENTIMENT(ID,CREATED_AT,TEXT,SEARCH_TEXT,SCREEN_NAME,AGG_SENTIMENT,AGG_SCORE) VALUES(?, ?, ?, ?, ?, ?, ?)";
		        Connection conn = VerticaDBUtil.getDBConnection();
		        PreparedStatement pstmt = conn.prepareStatement(sql);
		        pstmt.setLong(1, tweet.getId());
		        pstmt.setTimestamp(2, new java.sql.Timestamp(tweet.getCreatedAt().getTime()));
		        pstmt.setString(3, tweet.getText());
		        pstmt.setString(4, searchText);
		        pstmt.setString(5, tweet.getUser().getScreenName());
		        pstmt.setString(6, sentiment);
		        pstmt.setString(7, score);
		        VerticaDBUtil.beginTransaction();
		        int result = pstmt.executeUpdate();
		        if (result != 0) {
		        	VerticaDBUtil.commit();
		        } else {
		        	VerticaDBUtil.rollback();
		        }
		        VerticaDBUtil.closeDBUtil(null, pstmt, conn);
		        return result;
		    }
	}

- Create "IdolOnDemand.java"  
  Replace <API_KEY> with self key


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

	    private static String readToString(Reader rd) throws IOException {
	        StringBuilder sb = new StringBuilder();
	        char[] data = new char[1024];
	        int xf = 0;
	        while((xf = rd.read(data, 0, 1024)) > 0) {
	            sb.append(data, 0, xf);
	        }
	        return sb.toString();
	    }
	    
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

- Create "App.java". Make sure Twitter App Keys are feeded in PlaceHolders.


	import java.util.ArrayList;
	import java.util.List;

	import twitter4j.Query;
	import twitter4j.QueryResult;
	import twitter4j.Status;
	import twitter4j.Twitter;
	import twitter4j.TwitterException;
	import twitter4j.TwitterFactory;
	import twitter4j.auth.AccessToken;

	public class App {

		public static void main(String[] args) {
			TweetsUtil tUtil = new TweetsUtil();
			String tagName = "$HPQ";
		    Twitter twitter = TwitterFactory.getSingleton();
		    AccessToken accessToken = new AccessToken(, );
		    twitter.setOAuthConsumer(,);
		    twitter.setOAuthAccessToken(accessToken);

			try {
	 	    Query query = new Query(tagName);
		    QueryResult result;
		    List<Status> totalTweets = new ArrayList<Status>();
		        do {
					result = twitter.search(query);
	                List<Status> tweets = result.getTweets();
	                totalTweets.addAll(tweets);
				    for (Status tweet : tweets) {
	                    System.out.println("@" + tweet.getUser().getScreenName() + " - " + tweet.getText());
				    }
		        } while ((query = result.nextQuery()) != null);
		        tUtil.pushTweetsToDB(totalTweets,tagName);
	            System.exit(0);
			} catch (TwitterException te) {
	            te.printStackTrace();
	            System.out.println("Failed to search tweets: " + te.getMessage());
	            System.exit(-1);
			}
		}

	}

