# Welcome to Learn Progressive Web Apps!

### Foundations
All Progressive Web Apps are, at their core, modern websites, so it's important that your website has a solid foundation in responsive design, mobile and everything first, intrinsic design, and web performance.

### App design
One of the key differences between Progressive Web Apps and classic websites and web apps is installability. This creates a standalone experience more integrated into the platform and oeprating system. Installation enables new flexibility and new responsibility, as we won't have a browser's user interface around our content.

### Assets and data
A Progressive Web App is a website; all its assets are the same as on the web, but with new tools to make those assets load fast when online and available when offline.

### Service workers
Service workers are a fundamental part of a PWA. They enable fast loading (regardless of the network), offline access, push notifications, and other capabilities.

### Caching
You can use the Cache Storage API to download, store, delete or update assets on the device. Then these assets can be served on the device without needing a network request.

### Serving
Using the service worker's fetch event, you can intercept network requests and serve a response using different techniques.

### Workbox
Workbox is a set of modules that simplify common service worker interactions such as routing and caching. Each module addresses a specific aspect of service worker development. Workbox aims to make using service workers as easy as possible while allowing the flexibility to accommodate complex requirements where needed.

### Offline data
To build a solid offline experience you need to implement storage management. Tools like IndexedDB. Cache, Storage Manager, Persistent Storage, and Content Indexing can help.

### Installation
Installed apps are easy to access and can take advantage of some deeper integrations with the OS. Learn to make your PWA installable and gain those benefits.

### Web app manifest
The web app manifest is a JSON file that defines how the PWA should be treated as an installed application, including the look and feel and basic behavior within the operating system.

### Installation prompt 
For sites that pass the PWA install criteria, the browser triggers an event to prompt the user to install it. The good news is that you can use this event to customize your prompt and invite users to install your app.

### Update
Chances are your PWA needs updating. This chapter gives you the tools to update different parts of your PWA, from assets to metadata.

### Enhancements
Your users expects a good experience. In this chapter, you will see how to enhance your PWA with splash screens, app shortcuts, and how sessions work.

### Detection
Identifying how your users interact with your app is useful in customizing and improving the user experience. For example, you can check whether your app is already installed on the user's device and implement features such as transferring navigation to the standalone app from the browser.

### OS Integration
Your PWA now works outside the browser. This chapter covers how to integrate further with the operating system once users install your app.

### Window management
A PWA outside of the browser manages its own window. In this chapter, you will understand the APIs and capabilities for managing a window within the operating system.

### Experimental features
There are PWA capabilities that are still under construction and you can be part of their development. In this chapter you'll learn about the Fugu project, how to sign-up for an origin trial, and how to use experimentals APIs.

### Tools and debug
We will explore the tools available to develop, debug, and test your Progressive Web Apps.

### Architecture
You make some decisions when developing a PWA, such as whether to create a single page application or a multi-page application, and whether you will host it in the root of your domain or within  a folder.

### Complexity management
Keeping a web app simple can be surprisingly complicated. In this module, you will learn how web APIs work with threading and how you can use this for common PWA patterns such as state management.

### Capabilities
PWAs are not just tied to the screen. This chapter is about the capabilities that a PWA has today in terms of hardware, sensors, and platform usage.

# Progressive Web Apps
A Progressive Web App (PWA) is a web app that uses progressive enhancement to provide users with a more reliable experience, uses enw capabilities to provide a more integrated experience, and can be installed. And, because it's a web app, it can reach anyone, anywhere, on any device, all with a single codebase. Once installed, a PWA looks like any other app, specifically:
- It has an icon on the home screen, app launcher, launchpad, or start menu.
- It appears when you search for apps on the device.
- It opens in a standalone window, wholly separated from a browser's user interface.
- It has access to higher levels of integration with the OS, for example, URL handling or title bar customization. 
- It works offline.

### The web platform
The web is an incredible platform. Its mix of universality across devices and operating systems, its user-centered security model, and the fact that no single company controls its specification or implementation make it a powerful platform for delivering software.

Combined with the web's inherent linkability, it's possible to search across it and share what you've found with anyone, anywhere. Whenever you go to a website, it's the latest version the publisher deployed, and your experience with that site can be as temporary or as permanent as you'd like.

Web applications can reach anyone. anywhere, on any device with a single codebase. For developers, the web also offers a transparent and straightforward deploying mechanism. There is no need for packaging, no extra content review, or delays on updates. Users always get the latest version when they visit your app. With new capabilities and techniques, a web app can now allow you to interact or see content even when offline, a hurdle that was impossible to overcome a few years ago.

### Platform-specific apps
Platform-specific apps, on both mobile and desktop, are known for being rich and reliable. They're ever-present, on home screen, docks, and taskbars. They work regardless of network connection, and launch in their own standalone experience. They can read and write files from the local file system, access hardware connected via USB, serial, or Bluetooth, and interact with data stored on your devices, such as contacts and calendar events. In platform-specific applications, you can take pictures, play songs listed on the home screen, or control media playback while in another app. These applications feel like part of the device they run on.

> **Note:** In modern mobile operating systems, platform-specific apps are installed mostly from app stores, with rules and limitations on who can publish and what can be published for their users. These apps are typically shipped as a large, indivisible package, and every update needs re-packaging, re-signing, re-approval, and on-device re-installation.

A challenge for platform-specific apps is that they are not compatible with multiple platforms and devices, so it's not easy, if even possible, to move an Android app to an iOS to Windows or ChromeOS without creating a new app from scratch.

### Bringing the best of both worlds
If you think about platform apps and web apps in terms of capabilities and reach, platform apps represent the best of capabilities, whereas web apps represent the best of reach. Progressive Web Apps sit at the ntersection of the capabilities of platform apps and the reach of web apps. A Progressive Web App includes features from both worlds.

| Web | Platform apps |
| - | - | 
| - Linkability | - Offline-capable |
| - Accessible by default | - High performance |
| - Ubiquitous | - Device Integration |
| - Easy to deploy | - Standalone experience |
| - Easy to update | - Installed Icon |
| - Everyone can publish | - Rich and reliable |

> Note: People tend to think of Progressive Web Apps as an app that the user can install from a browser instead of an app store. However, a PWA can be listed in many app stores today as an optional distribution channel, including Google Play Store (for Android and ChromeOS), Microsoft Store (for Windows 10 and 11), and Apple AppStore (for iOS, iPadOS, and macOS). For these cases, you must follow all store rules and requirements, but you will still get some of the advantages of a PWA.

### Adoption has its benefits
Hulu, a video streaming service in the USA, created a Progressive Web App version of their experience to replace their desktop apps which had poor user reviews and poor usage. As shared as Google I/O 2019, one developer could research and implement this experience from their existing web application in two weeks.

Within five months, 96% of their legacy app users had adopted the PWA, with a 27% increase in return visits and a 5.5% increase in engagement. Because it's in the launcher and on taskbars, PWAs are easier to return to than if they just lived in a tab.

JD.ID, an e-commerce platform in Indonesia providing delivery services for many products, wanted to expand its online presence by focusing on performance and a network-independent solid experience for their PWA. With this enhanced experience, they increased their overall mobile conversion rate by 53%, 200% for installed users, and increased their daily active users by 26%.

Clipchamp is an in-browser, desktop-class online video editor that empowers anyone to tell stories worth sharing through video. They saw 9% higher user retention with their PWA versus their standard desktop app users and have seen their PWA installation at rate of 97% each month in its first five months launched.

Corel Corporation's Gravit Designer is a powerful, desktop-class vector design tool that serves tens of thousands of daily active users demanding rich, affordable, accessible vector illustration software. Since adding a PWA as an install option for users, they've seen PWA users are 24% more active, the PWA accounts for 31% more repeat users, and PWA users are 2.5 times more likely to purchase Gravit Designer PRO, as compared to their other platforms and install options.

> Note: Many other companies have implemented PWA and seen a benefit. Large companies have already published PWAs on various products, including Apple (AppStore Connect, Feedback Assistant), Microsoft (Office 365, Windows 365), Google (Duo, YouTube Music, Drive), Amazon (Luna), Facebook (Instagram Lite, Gaming).

#### The streaming game changer

A great example of the power of Progressive Web Apps is the industry of streaming platforms, including cloud gaming and remote computing. Since 2021, most cloud game providers have launched Progressive Web Apps, letting you play console games from any device and just a browser or a PWA installation: iPhone, Android, iPad, laptops, Macs, or PCs. Amazon Luna, Microsoft Xbox Cloud Gaming, Facebook Gaming, Google Stadia, Nvidia GeForce Now, and BlueStacks X offer cloud gaming solutions over the browser as PWAs. They all provide a great experience with performance close to native on all platforms thanks to web technologies such as WebRTC, WebAssembly, and GamePad APIs.

### Challenges
Having covered the advantages of using the web platform to publish PWAs, it's also important to be aware of the challenges you may face.

#### Cross-browser compatibility
Apple is a crucial company for the multi-device world, owning iOS, iPadOS, macOS, and Safari. While Apple has never used the term PWA in public, they've been supporting the technologies to make a PWA installable and offline-capable since 2018 on Safari for iPhones and iPads.

However, Apple's implementation of the PWA specs misses many features possessed by other browsers, in particular browsers powered by the Chromium engine.

In the middle, we also have Firefox and its Gecko engine with implementations including more PWA specs on Android, and fewer installation capabilities on desktop.

Limitations include the lack of push notifications, integration APIs (such as Web Bluetooth or WebNFC), and installation promotion techniques that help users know they can install the current website to get an app experience. In addition, there are several bugs with implemented features.

As with all web development, testing your experience on every platform is mandatory when releasing your PWA, and when a major new browser or OS version is released. You should always provide fallback solutions or alternative experiences when a feature is not available.

#### Awareness of PWAs
As a PWA developer, you will probably encounter an awareness problem, both on the business and user sides. Some business owners won't know about PWAs or will have misconceptions about the power and challenges of Progressive Web Apps.

When you publish a PWA, your next challenge is ensuring users understand that the website is installable, leading to an installed app experience.

The installation challenge is more significant on some platforms, such as iOS and iPadOS, and sometimes UX designers include screens that explain to the user how to install the app.

### Compatibility
You need to remember that a Progressive Web App is just a web app, so content and services are running on top of standard specs and protocols. Therefore, a PWA technically runs everywhere the web runs; you don't need the platform to be compatible with any "PWA spec."

Ho