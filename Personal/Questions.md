Hi A.Ahad,

I reviewed your application for our Working Student Mobile App Developer (f/m/d), KI, AI job and am impressed with your background. 

I'd like to learn a bit more about your work. Please share brief answers to these questions:

## 1) Please briefly describe the most complex iOS feature you've implemented in production. What made it challenging?

  **Feature**
  
When I worked as part of Digital Dividend, we had to create a single server-driven UI education-based white-labeled app with generic multi-modules, such as book learning, video learning, immersive learning with games, etc. The idea was to sell this app to as many clients as needed, customized to their own UI and whichever features they wanted from those made available to them, while keeping the entire codebase the same. **50+** apps were created from this codebase. You can find the list of apps [here](https://www.thelearningapps.com/all-apps/). It was very complex app with lots of challenges but the following were which i found the most challenging. 
  

  **Challenges**:
  
   ***1- Server driven UI:***
    In order to achieve dynamic UI rendering, real-time changes, and platform-agnostic designing, creating a schema or model for various components in JSON was a very challenging       task that required extensive communication and iteration to get it right and stable, not just on the client side but also on the server side. On the client side, challenges         such as complex parsing and performance overhead were also considerations, requiring significant threading knowledge to make processes faster.

   ***2-Scalability and Maintainance:***
    The codebase became heavily coupled with platform-specific features, brand customizations, and conditional logic for multiple apps, leading to a codebase that was hard to   
    maintain. Changes in shared code for bug fixes sometimes had unintended side effects on certain apps. I had to create many independent, generic modules with customizations 
    injected through dependency injection and plugin-based systems, using a main module where all other modules were composed into a single app with server-side JSON coming in
    real-time. A configuration management system was also put in place, utilizing configuration files for app-specific settings, branding, and features. A lot of iterations were 
    involved before any stability was seen.
    
   ***3-Testing and Deployment:***
    Writing and maintaining automated tests for all possible configurations and customizations became very cumbersome.Making sure changes in shared code don't break functionality 
    for any other app required a lot of integeration testing with its own individual test plan not to mention writing and maintaining tests for independent modules.
    **Deployment** for so many apps was very challenging because each app had unique metadata, assets, and requirements for app store submission, increasing the effort required for 
    deployment. Not to mention managing separate build pipelines for 50+ apps. We had a custom CI/CD pipeline, and I added a custom script for the marketing team that provided a 
    popup where they could choose which apps to deploy in parallel, saving both development time and CI/CD costs.
    

   
  There were also other challenges like Resource Allocation, Performance , Security Documentation etch

    
    
## 2) When you discover a critical bug in production, what are your first 3 steps?

1. **Reproduce the bug**:  
   This could be through analyzing logs, testing different app configurations or OS versions,
   reviewing user feedback reports, or utilizing crash monitoring services like Crashlytics or Sentry. 
   Gathering as much information as possible to reproduce the bug locally and identify its root cause is crucial.
   Additionally, using tools like `git bisect` to isolate the commit or branch that introduced the bug can be incredibly helpful, in my opinion.

2. **Fix, test, and verify**:  
   The **fix** could involve writing a patch or even clean refactoring depending on whether a lot of coupled dependencies are involved.
   However, the main focus should be on adding tests to validate the fix, which could be unit tests or even integration tests depending on the fix. 
   Verifying the issue with testing on real devices or conducting regression testing locally or in a staging environment with real-world simulation also helps in validation.

3. **Release and monitor and future prevention**:  
   The release could be a hotfix if it requires immediate deployment, or the code could be submitted for review and QA testing if it doesn't. 
   After deployment to the app store, we can monitor post-release crash rates, logs, or user feedback to determine whether the fix was successful.
   In the future, preventive measures like improving tests, CI/CD pipelines, or code reviews can be added. 
   Updating documentation or coding guidelines or writing a postmortem to share findings and solutions can also be beneficial.


## 3) if you had 2 weeks to build an MVP for a basic photo sharing app, what would be your technical choices and priorities? Please outline your approach in 3-4 points

For the **MVP**, the focus is on going as fast as possible and delivering within the timeframe.
Taking that into consideration, we don't want to burden ourselves with architecture, testing, modularization, analytics, monitoring, or having processes such as CI/CD, code review, or QA testing in place.
These are tech debts that we are willing to take; however, in my opinion, even with a monolith, we need to keep our code decoupled enough that we can leverage dependency injection/inversion and composition so that,
in the future, we can scale if needed.

For the **tech stack**,
it depends firstly on whether we need the app on both platforms and if our future requirements envisage a single codebase for both platforms.
Additionally, we need to consider the expertise of the current team. 
In a hybrid approach, using Flutter or React Native would suit better; however, if we want to focus on a single platform like iOS, for example, then we have the liberty to choose between two UI frameworks: UIKit and SwiftUI.
SwiftUI, with its declarative paradigm, would help us move faster. 

For the **backend side**, we can leverage backend-as-a-service platforms like Firebase, Supabase, or Appwrite, where we don't have to write a backend ourselves, saving a lot of time.
Having said that, we need to be aware of the costs these services impose, as they can become significant if things scale.
We need to write decoupled code in a way that allows us to easily swap these services in the future with our own backend if needed, without affecting other parts of the app.

Having said the above my approach would be as follows.

 1. **Define Core Features ( Prioritize Scope)**

	•	***User Authentication:***  users can sign up, log in, and manage sessions.We can Use Firebase Authentication for simplicity and speed.

	•	***Photo Upload & Storage:*** Enable users to upload photos with basic metadata (e.g., captions, timestamps). Use a managed storage solution like Firebase Storage or AWS S3.

	•	***Feed Display:*** Show a basic photo feed with user-uploaded images in reverse chronological order.

	•	***Basic Interactions:*** Add the ability to “like” photos or leave comments.
2. **Technology Stack**

	***•	Frontend (iOS):***
	  •	Use SwiftUI for rapid UI development and declarative layouts.
	  •	Integrate Combine or async/await for handling asynchronous data flows.
   
	***•	Backend:***
	  •	Use Firebase (Firestore or Realtime Database) for a serverless backend to handle user data, photo metadata, and likes/comments.
	  •	Optional: Use Cloud Functions for any backend logic (e.g., notifications, feed sorting).
   
	***•	Storage:***
	  •	Firebase Storage for quick setup of secure, scalable image hosting.

3. **Development Plan**

	**•	Week 1:**
   
	 ***1.	Set Up Project:***
	    •	Create the app project and configure Firebase for authentication, Firestore, and storage.
   
	 ***2.	Implement Authentication:***
	    •	Add email/password and optional social logins.
   
   ***3.	Build the Photo Feed:***
    	•	Design and implement a photo feed using a LazyVStack in SwiftUI.
   

	**•	Week 2:**
   
	 ***1.	Photo Upload:***
	   •	Integrate the image picker for users to select and upload photos.
	   •	Save metadata to Firestore and images to Firebase Storage.

	 ***2.	Add Interactions:***
	   •	Implement basic likes and comments functionality with Firestore.
   
	 ***3.	Testing & Polishing:***
	   •	Test the app on multiple devices, resolve bugs, and refine the UI/UX.

4. **Deployment & Scalability**

	  •	***Testing:***:
      • Use TestFlight for beta distribution and gather user feedback.
   
	  •	***Future Scalability:***:
      • Ensure scalable architecture (e.g., MVVM or Clean Architecture) for easier feature expansion.	Use dependency injection for loosely coupled components to accommodate future backend changes

## 4) Whats your biggest strength

I think i can divide this into 3 categories:

  1. **Technical**:
     
     ***Strong Mobile Development Foundation***
      I believe I have a strong core foundation in mobile development, which I can transfer irrespective of platforms.
      This includes requirements gathering, ecosystem understanding, architecture, modularization, testing, design principles, code quality, CI/CD, code reviews, and deployment.
 
     ***Adaptability to Tools and Technologies***
      I can quickly learn new tools, libraries, and technologies. My mobile development tech stack includes native iOS, Flutter, and React Native.
      I also do front-end web development with React and back-end development with Node.js, and I am always willing to learn new technologies.

     ***Code Quality, Testing and Scalability***
      My focus in recent years has been on writing clean, maintainable, and well-documented code.
      As part of the process, I typically follow TDD, clean architecture, and dependency injection.
     
  3. **Social**:
     
      ***Growth Mindset***
       I have a growth mindset.I always strive to learn and improve every day in any way possible. Learning never stops.
     
     ***Communication Skills***
       I have worked extensively on my communication skills to clearly articulate ideas, technical concepts, and solutions, as well as actively listen to the team and understand           their needs.

     ***Adaptability***
       Adaptability is one of my biggest assets. I can work independently or as part of cross-functional teams (developers, designers, product managers, etc.).


4. **Ownership**:
     
     ***Accountability***
         i take full responsibility of my work to deliver on time and make sure i follow through on commitments and owning mistakes when they occur.
      

## 5) Could you please provide a 5-10 minute screen recording of one of your codebase, walking through explaining the technical architecture?

**here is the video:**
https://drive.google.com/file/d/16by4D4VFCNbpcPp6VL3i8hNMtENH2pwa/view?usp=sharing


**UIKit+MVP+Clean architecture codebase**
https://github.com/abdahad1996/EssentialFeed/tree/main

**swiftUI+MVVM+Clean architecture codebase**
https://github.com/abdahad1996/Img.ly


**You can access here all my technical knowledge in iOS**
https://me.essentialdeveloper.com/achievements/b9adeee4-eee8-4516-97e9-925704d8b752
