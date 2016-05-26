---
layout: post
title: Firebase Mobile Analytics First Experiences
date: '2016-05-25T12:00:00.000-08:00'
author: David Green
tags:
  - Android
  - Analytics
  - Firebase
modified_time: '2016-05-25T12:00:00.000-08:01'
comments: true
---

More information is better when it comes to improving any application.  With a mission to find out how people are using a new Android app that I'm working on, I set out to integrate analytics into my mobile app.  Imagine my excitement to discover that [Google has launched Firebase Analytics](https://searchenginewatch.com/2016/05/19/google-launches-firebase-analytics-for-mobile-apps/) for mobile apps at Google I/O 2016. It took me all of about 20 minutes to get the initial analytics integration working.  Here's what I did:

#### 1. Setup a Firebase Account

The first step is to [create a Firebase account](https://firebase.google.com/).  This was really simple using my Google credentials.  Firebase was able to create a new analytics project for my existing mobile application project.  To do that I selected "Import Google Project" and provided the package name (found in `AndroidManifest.xml`), and the SHA1 certificate fingerprint from my signing digital cert.  The fingerprint can be obtained with a command like this one:

     $ keytool -list -v -keystore ~/my-keystore.jks
     ...snip..

     Alias name: alias_name
     Creation date: 25-May-2016
     Entry type: PrivateKeyEntry
     Certificate chain length: 1
     Certificate[1]:
     Owner: CN=David Green, OU=Unknown, O=David Green, L=Vancouver, ST=British Columbia, C=CA
     Issuer: CN=David Green, OU=Unknown, O=David Green, L=Vancouver, ST=British Columbia, C=CA
     Serial number: 60eef983
     Valid from: Wed May 25 21:02:06 PDT 2016 until: Sun Oct 11 21:02:06 PDT 2043
     Certificate fingerprints:
     	 MD5:  24:4C:92:9B:0D:01:26:E0:1A:49:DF:01:8F:3B:63:A0
     	 SHA1: 5B:16:23:8A:A0:30:8D:B6:B6:7F:7B:96:EE:F3:10:6C:95:EF:0D:9F
     	 SHA256: 2E:0E:29:D2:67:9D:12:65:46:C4:F5:46:A0:8B:D8:8F:FE:3C:A9:C5:6B:E4:A2:8E:FA:01:18:3E:6A:82:AE:37
     	 Signature algorithm name: SHA256withRSA
     	 Version: 3

#### 2. Add Firebase Dependencies

Next was adding Firebase dependencies to the build.  [The instructions](https://firebase.google.com/docs/android/setup#add_the_sdk) are great, it took all of 2 minutes.  The only hiccup is that I had to update an existing project dependency `com.google.android.gms:play-services-maps` to the 9.0.0 version to get the build to work.

#### 3. Add Analytics To The App

Finally adding analytics to the app was trivial.  A key step for me is that this feature has to be opt-in.  In other words, the user has to agree to providing stats, otherwise it just doesn't feel right.  To make it easy to manage the opt-in setting I wrapped the Firebase APIs in a small class called `Analytics` as follows:

    public class Analytics {

    	private static final String CONTENT_TYPE_ACTIVITY = "activity";

    	static final String ENABLED_KEY = "enableAnalytics";

    	private boolean enabled;
    	private FirebaseAnalytics firebase;

    	public Analytics(Context context) {
    		checkNotNull(context);
    		this.enabled = PreferenceManager.getDefaultSharedPreferences(context).getBoolean(ENABLED_KEY, false);
    		if (enabled) {
    			firebase = FirebaseAnalytics.getInstance(context);
    		}
    	}

    	public void logActivated(Activity activity) {
    		if (!enabled) {
    			return;
    		}
    		Bundle bundle = new Bundle();
    		bundle.putString(FirebaseAnalytics.Param.ITEM_ID, activity.getClass().getSimpleName());
    		bundle.putString(FirebaseAnalytics.Param.CONTENT_TYPE, CONTENT_TYPE_ACTIVITY);
    		firebase.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT, bundle);
    	}
    }

To make this work with all of the activities in my app, I created an Activity base class as follows:

    public class AnalyticsActivity extends AppCompatActivity {
    	protected Analytics analytics;

    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
    		super.onCreate(savedInstanceState);
    		analytics = new Analytics(this);
    	}

    	@Override
    	protected void onResume() {
    		super.onResume();
    		analytics.logActivated(this);
    	}
    }

Finally with a little search-and-replace, I had all activity classes extending `AnalyticsActivity` instead of `AppCompatActivity`.  

That's it!  With just this mobile analytics will help me to understand how much and how often each activity within the app is used in much the same way as Google Analytics does the same for the web.  Now only if creating a privacy policy was as straight forward!
