---
title: "Google Assistant Development for Android Devs (Part 2)"
author: Patrick Jackson
categories: [Development, Android]
tags: [Google Assistant, Actions on Google, conversational interfaces, Kotlin]
---


![](https://storage.googleapis.com/kotlin-actions-sdk.appspot.com/actions-kotlin-java.png)

This is part 2 in a series on developing Actions on Google.  [Part one is here](http://ticketmastermobilestudio.com/blog/google-assistant-development-for-android-devs-part-1).

In my last post, I introduced an open source SDK for Actions on Google for Kotlin and Java.  Today, that library port is complete, with all features supported.  

 - API.AI support
 - Actions SDK support
 - All tests ported and passing (~230)
 - Conversation components example for API.AI & Actions SDK
 - Transaction Sample for API.AI
 - Java conversation sample
 
Now the capabilities of the [official Node.js SDK](https://github.com/actions-on-google/actions-on-google-nodejs){:target='_blank'} can be matched easily by using the [unofficial Kotlin/Java SDK](https://github.com/TicketmasterMobileStudio/actions-on-google-kotlin).  Since this is a port, and the API is nearly identical, one can look at the Node.js docs and examples and easily do the same in Kotlin, Java, or any JVM language.  Below is how writing Actions for Google with the unofficial SDK looks.

### Actions in Kotlin

{% raw %}
```kotlin
    fun welcome(app: ApiAiApp) =
        app.ask(app.buildRichResponse()
                .addSimpleResponse(speech = "Hi there!", displayText = "Hello there!")
                .addSimpleResponse(
                        speech = """I can show you basic cards, lists and carousels as well as
                    "suggestions on your phone""",
                        displayText = """"I can show you basic cards, lists and carousels as
                    "well as suggestions"""")
                .addSuggestions("Basic Card", "List", "Carousel", "Suggestions"))
                
    fun normalAsk(app: ApiAiApp) = app.ask("Ask me to show you a list, carousel, or basic card")

    fun suggestions(app: ApiAiApp) {
        app.ask(app
            .buildRichResponse()
            .addSimpleResponse("This is a simple response for suggestions")
            .addSuggestions("Suggestion Chips")
            .addSuggestions("Basic Card", "List", "Carousel")
            .addSuggestionLink("Suggestion Link", "https://assistant.google.com/"))
    }
    
    val actionMap = mapOf(
        WELCOME to ::welcome,
        NORMAL_ASK to ::normalAsk,
        SUGGESTIONS to ::suggestions)
      
    
    @WebServlet("/conversation")
    class WebHook : HttpServlet() {

    	override fun doPost(req: HttpServletRequest, resp: HttpServletResponse) {
        	ApiAiAction(req, resp).handleRequest(actionMap)
       }
    }
```
{% endraw %}

### Actions in Java

{% raw %}
```java
	@WebServlet("/conversation/java")
	public class ConversationComponentsSampleJava extends HttpServlet {
    	private static final Logger logger = Logger.getAnonymousLogger();

		Function1<ApiAiApp, Object> welcome = app -> {
        	app.ask(app.buildRichResponse()
                .addSimpleResponse("Hi there from Java!", "Hello there from Java!")
                .addSimpleResponse(
                        "I can show you basic cards, lists and carousels as well as suggestions on your phone",
                        "I can show you basic cards, lists and carousels as well as suggestions")
                .addSuggestions("Basic Card", "List", "Carousel", "Suggestions"), null);
        	return Unit.INSTANCE;
    	};

    	Function1<ApiAiApp, Object> normalAsk = app ->
       	     app.ask("Ask me to show you a list, carousel, or basic card");

    	Function1<ApiAiApp, Object> suggestions = app ->
       	     app.ask(app.buildRichResponse(null)
                    .addSimpleResponse("This is a simple response for suggestions", null)
                    .addSuggestions("Suggestion Chips")
                    .addSuggestions("Basic Card", "List", "Carousel")
                    .addSuggestionLink("Suggestion Link", "https://assistant.google.com/"));

		private Map<String, Function1<String, Object>> intentMap = new HashMap() {{
        	put(ConversationComponentsSampleKt.WELCOME, welcome);
        	put(ConversationComponentsSampleKt.NORMAL_ASK, normalAsk);
        	put(ConversationComponentsSampleKt.SUGGESTIONS, suggestions);
    	}};

    	@Override
    	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         	 ApiAiAction action = new ApiAiAction(req, resp);
       	 	 action.handleRequest(intentMap);
    	}
    }
```
{% endraw %}

 
Many Android and Java developers should be able to get started quickly, even if your're a newbie to back-end development.  Check out [our GitHub repo]((https://github.com/TicketmasterMobileStudio/actions-on-google-kotlin) for examples and more information on getting started.

In the next post I'll be covering some of the basic concepts and tools for developing Actions on Google.

