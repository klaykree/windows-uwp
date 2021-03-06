---
author: TylerMSFT
title: Display a splash screen for more time
description: Display a splash screen for more time by creating an extended splash screen for your app. This extended screen imitates the splash screen shown when your app is launched, but can be customized.
ms.assetid: CD3053EB-7F86-4D74-9C5A-950303791AE3
ms.author: twhitney
ms.date: 02/08/2017
ms.topic: article
ms.prod: windows
ms.technology: uwp
keywords: windows 10, uwp
---

# Display a splash screen for more time




**Important APIs**

-   [**SplashScreen class**](https://msdn.microsoft.com/library/windows/apps/br224763)
-   [**Window.SizeChanged event**](https://msdn.microsoft.com/library/windows/apps/br209055)
-   [**Application.OnLaunched method**](https://msdn.microsoft.com/library/windows/apps/br242335)

Display a splash screen for more time by creating an extended splash screen for your app. This extended screen imitates the splash screen shown when your app is launched, but can be customized. Whether you want to show real-time loading information or simply give your app extra time to prepare its initial UI, an extended splash screen lets you define the launch experience.

> **Note**  The phrase "extended splash screen" in this topic refers to a splash screen that stays on the screen for an extended period of time. It does not mean a subclass that derives from the [**SplashScreen**](https://msdn.microsoft.com/library/windows/apps/br224763) class.

 

Make sure your extended splash screen accurately imitates the default splash screen by following these recommendations:

-   Your extended splash screen page should use a 620 x 300 pixel image that is consistent with the image specified for your splash screen in your app manifest (your app's splash screen image). In Microsoft Visual Studio 2015, splash screen settings are stored in the **Splash Screen** section of the **Visual Assets** tab in your app manifest (Package.appxmanifest file).
-   Your extended splash screen should use a background color that is consistent with the background color specified for your splash screen in your app manifest (your app's splash screen background).
-   Your code should use the [**SplashScreen**](https://msdn.microsoft.com/library/windows/apps/br224763) class to position your app's splash screen image at the same screen coordinates as the default splash screen.
-   Your code should respond to window resize events (such as when the screen is rotated or your app is moved next to another app onscreen) by using the [**SplashScreen**](https://msdn.microsoft.com/library/windows/apps/br224763) class to reposition items on your extended splash screen.

Use the following steps to create an extended splash screen that effectively imitates the default splash screen.

## Add a **Blank Page** item to your existing app


This topic assumes you want to add an extended splash screen to an existing Universal Windows Platform (UWP) app project using C#, Visual Basic, or C++.

-   Open your app in Visual Studio 2015.
-   Press or open **Project** from the menu bar and click **Add New Item**. An **Add New Item** dialog box will appear.
-   From this dialog box, add a new **Blank Page** to your app. This topic names the extended splash screen page "ExtendedSplash".

Adding a **Blank Page** item generates two files, one for markup (ExtendedSplash.xaml) and another for code (ExtendedSplash.xaml.cs).

## Essential XAML for an extended splash screen


Follow these steps to add an image and progress control to your extended splash screen.

In your ExtendedSplash.xaml file:

-   Change the [**Background**](https://msdn.microsoft.com/library/windows/apps/br209396) property of the default [**Grid**](https://msdn.microsoft.com/library/windows/apps/br242704) element to match the background color you set for your app's splash screen in your app manifest (in the **Visual Assets** section of your Package.appxmanifest file). The default splash screen color is a light gray (hex value \#464646). Note that this **Grid** element is provided by default when you create a new **Blank Page**. You don't have to use a **Grid**; it's just a convenient base for building an extended splash screen.
-   Add a [**Canvas**](https://msdn.microsoft.com/library/windows/apps/br209267) element to the [**Grid**](https://msdn.microsoft.com/library/windows/apps/br242704). You'll use this **Canvas** to position your extended splash screen image.
-   Add an [**Image**](https://msdn.microsoft.com/library/windows/apps/br242752) element to the [**Canvas**](https://msdn.microsoft.com/library/windows/apps/br209267). Use the same 600 x 320 pixel image for your extended splash screen that you chose for the default splash screen.
-   (Optional) Add a progress control to show users that your app is loading. This topic adds a [**ProgressRing**](https://msdn.microsoft.com/library/windows/apps/br227538), instead of a determinate or indeterminate [**ProgressBar**](https://msdn.microsoft.com/library/windows/apps/br227529).

Add the following code to define [**Canvas**](https://msdn.microsoft.com/library/windows/apps/br209267) and [**Image**](https://msdn.microsoft.com/library/windows/apps/br242752) elements, as well as a [**ProgressRing**](https://msdn.microsoft.com/library/windows/apps/br227538) control, in ExtendedSplash.xaml:

```xml
    <Grid Background="#464646">
        <Canvas>
            <Image x:Name="extendedSplashImage" Source="Assets/SplashScreen.png"/>
            <ProgressRing Name="splashProgressRing" IsActive="True" Width="20" HorizontalAlignment="Center"></ProgressRing>
        </Canvas>
    </Grid>
```

**Note**  This code sets the width of the [**ProgressRing**](https://msdn.microsoft.com/library/windows/apps/br227538) to 20 pixels. You can manually set its width to a value that works for your app, however, the control will not render at widths of less than 20 pixels.

 

## Essential code for an extended splash screen class


Your extended splash screen needs to respond whenever the window size (Windows only) or orientation changes. The position of the image you use must be updated so that your extended splash screen looks good no matter how the window changes.

Use these steps to define methods to correctly display your extended splash screen.

1.  **Add required namespaces**

    You'll need to add the following namespaces to ExtendedSplash.xaml.cs to access the [**SplashScreen**](https://msdn.microsoft.com/library/windows/apps/br224763) class, [**Window.SizeChanged**](https://msdn.microsoft.com/library/windows/apps/br209055) events.

    ```cs
    using Windows.ApplicationModel.Activation;
    using Windows.UI.Core;
    ```

2.  **Create a partial class and declare class variables**

    Include the following code in ExtendedSplash.xaml.cs to create a partial class to represent an extended splash screen.

    ```cs
    partial class ExtendedSplash : Page
    {
        internal Rect splashImageRect; // Rect to store splash screen image coordinates.
        private SplashScreen splash; // Variable to hold the splash screen object.
        internal bool dismissed = false; // Variable to track splash screen dismissal status.
        internal Frame rootFrame;

       // Define methods and constructor
    }
    ```

    These class variables are used by several methods. The `splashImageRect` variable stores the coordinates where the system displayed the splash screen image for the app. The `splash` variable stores a [**SplashScreen**](https://msdn.microsoft.com/library/windows/apps/br224763) object, and the `dismissed` variable tracks whether or not the splash screen that is displayed by the system has been dismissed.

3.  **Define a constructor for your class that correctly positions the image**

    The following code defines a constructor for the extended splash screen class that listens for window resizing events, positions the image and (optional) progress control on the extended splash screen, creates a frame for navigation, and calls an asynchronous method to restore a saved session state.

    ```cs
    public ExtendedSplash(SplashScreen splashscreen, bool loadState)
    {
        InitializeComponent();

        // Listen for window resize events to reposition the extended splash screen image accordingly.
        // This ensures that the extended splash screen formats properly in response to window resizing.
        Window.Current.SizeChanged += new WindowSizeChangedEventHandler(ExtendedSplash_OnResize);

        splash = splashscreen;
        if (splash != null)
        {
            // Register an event handler to be executed when the splash screen has been dismissed.
            splash.Dismissed += new TypedEventHandler<SplashScreen, Object>(DismissedEventHandler);

            // Retrieve the window coordinates of the splash screen image.
            splashImageRect = splash.ImageLocation;
            PositionImage();

            // If applicable, include a method for positioning a progress control.
            PositionRing();
        }

        // Create a Frame to act as the navigation context
        rootFrame = new Frame();            
    }
    ```

    Make sure to register your [**Window.SizeChanged**](https://msdn.microsoft.com/library/windows/apps/br209055) handler (`ExtendedSplash_OnResize` in the example) in your class constructor so that your app positions the image correctly in your extended splash screen.

4.  **Define a class method to position the image in your extended splash screen**

    This code demonstrates how to position the image on the extended splash screen page with the `splashImageRect` class variable.

    ```cs
    void PositionImage()
    {
        extendedSplashImage.SetValue(Canvas.LeftProperty, splashImageRect.X);
        extendedSplashImage.SetValue(Canvas.TopProperty, splashImageRect.Y);
        extendedSplashImage.Height = splashImageRect.Height;
        extendedSplashImage.Width = splashImageRect.Width;
    }
    ```

5.  **(Optional) Define a class method to position a progress control in your extended splash screen**

    If you chose to add a [**ProgressRing**](https://msdn.microsoft.com/library/windows/apps/br227538) to your extended splash screen, position it relative to the splash screen image. Add the following code to ExtendedSplash.xaml.cs to center the **ProgressRing** 32 pixels below the image.

    ```cs
    void PositionRing()
    {
        splashProgressRing.SetValue(Canvas.LeftProperty, splashImageRect.X + (splashImageRect.Width*0.5) - (splashProgressRing.Width*0.5));
        splashProgressRing.SetValue(Canvas.TopProperty, (splashImageRect.Y + splashImageRect.Height + splashImageRect.Height*0.1));
    }
    ```

6.  **Inside the class, define a handler for the Dismissed event**

    In ExtendedSplash.xaml.cs, respond when the [**SplashScreen.Dismissed**](https://msdn.microsoft.com/library/windows/apps/br224764) event occurs by setting the `dismissed` class variable to true. If your app has setup operations, add them to this event handler.

    ```cs
    // Include code to be executed when the system has transitioned from the splash screen to the extended splash screen (application's first view).
    void DismissedEventHandler(SplashScreen sender, object e)
    {
        dismissed = true;

        // Complete app setup operations here...
    }
    ```

    After app setup is complete, navigate away from your extended splash screen. The following code defines a method called `DismissExtendedSplash` that navigates to the `MainPage` defined in your app's MainPage.xaml file.

    ```cs
    async void DismissExtendedSplash()
      {
         await Windows.ApplicationModel.Core.CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal,() =>            {
              rootFrame = new Frame();
              rootFrame.Content = new MainPage(); Window.Current.Content = rootFrame;
            });
      }
      ```

7.  **Inside the class, define a handler for Window.SizeChanged events**

    Prepare your extended splash screen to reposition its elements if a user resizes the window. This code responds when a [**Window.SizeChanged**](https://msdn.microsoft.com/library/windows/apps/br209055) event occurs by capturing the new coordinates and repositioning the image. If you added a progress control to your extended splash screen, reposition it inside this event handler as well.

    ```cs
    void ExtendedSplash_OnResize(Object sender, WindowSizeChangedEventArgs e)
    {
        // Safely update the extended splash screen image coordinates. This function will be executed when a user resizes the window.
        if (splash != null)
        {
            // Update the coordinates of the splash screen image.
            splashImageRect = splash.ImageLocation;
            PositionImage();

            // If applicable, include a method for positioning a progress control.
            // PositionRing();
        }
    }
    ```

    **Note**  Before you try to get the image location make sure the class variable (`splash`) contains a valid [**SplashScreen**](https://msdn.microsoft.com/library/windows/apps/br224763) object, as shown in the example.

     

8.  **(Optional) Add a class method to restore a saved session state**

    The code you added to the [**OnLaunched**](https://msdn.microsoft.com/library/windows/apps/br242335) method in Step 4: [Modify the launch activation handler](#modify-the-launch-activation-handler) causes your app to display an extended splash screen when it launches. To consolidate all methods related to app launch in your extended splash screen class, you could consider adding an asynchronous method to your ExtendedSplash.xaml.cs file to restore the app's state.

    ```cs
    async void RestoreStateAsync(bool loadState)
    {
        if (loadState)
        {
             // code to load your app's state here
        }
    }
    ```

    When you modify the launch activation handler in App.xaml.cs, you'll also set `loadstate` to true if the previous [**ApplicationExecutionState**](https://msdn.microsoft.com/library/windows/apps/br224694) of your app was **Terminated**. If so, the `RestoreStateAsync` method restores the app to its previous state. For an overview of app launch, suspension, and termination, see [App lifecycle](app-lifecycle.md).

## Modify the launch activation handler


When your app is launched, the system passes splash screen information to the app's launch activation event handler. You can use this information to correctly position the image on your extended splash screen page. You can get this splash screen information from the activation event arguments that are passed to your app's [**OnLaunched**](https://msdn.microsoft.com/library/windows/apps/br242335) handler (see the `args` variable in the following code).

If you have not already overridden the [**OnLaunched**](https://msdn.microsoft.com/library/windows/apps/br242335) handler for your app, see [App lifecycle](app-lifecycle.md) to learn how to handle activation events.

In App.xaml.cs, add the following code to create and display an extended splash screen.

```cs
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    if (args.PreviousExecutionState != ApplicationExecutionState.Running)
    {
        bool loadState = (args.PreviousExecutionState == ApplicationExecutionState.Terminated);
        ExtendedSplash extendedSplash = new ExtendedSplash(args.SplashScreen, loadState);
        Window.Current.Content = extendedSplash;
    }

    Window.Current.Activate();
}
```

## Complete code


> **Note**  The following code slightly differs from the snippets shown in the previous steps.
-   ExtendedSplash.xaml includes a `DismissSplash` button. When this button is clicked, an event handler, `DismissSplashButton_Click`, calls the `DismissExtendedSplash` method. In your app, call `DismissExtendedSplash` when your app is done loading resources or initializing its UI.
-   This app also uses a UWP app project template, which uses [**Frame**](https://msdn.microsoft.com/library/windows/apps/br242682) navigation. As a result, in App.xaml.cs, the launch activation handler ([**OnLaunched**](https://msdn.microsoft.com/library/windows/apps/br242335)) defines a `rootFrame` and uses it to set the content of the app window.

ExtendedSplash.xaml: This example includes a `DismissSplash` button because it doesn't have app resources to load. In your app, dismiss the extended splash screen automatically when your app is done loading resources or preparing its initial UI.

```xml
<Page
    x:Class="SplashScreenExample.ExtendedSplash"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:SplashScreenExample"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="#464646">
        <Canvas>
            <Image x:Name="extendedSplashImage" Source="Assets/SplashScreen.png"/>
            <ProgressRing Name="splashProgressRing" IsActive="True" Width="20" HorizontalAlignment="Center"/>
        </Canvas>
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Bottom">
            <Button x:Name="DismissSplash" Content="Dismiss extended splash screen" HorizontalAlignment="Center" Click="DismissSplashButton_Click" />
        </StackPanel>
    </Grid>
</Page>
```

ExtendedSplash.xaml.cs: Note that the `DismissExtendedSplash` method is called from the click event handler for the `DismissSplash` button. In your app, you won't need a `DismissSplash` button. Instead, call `DismissExtendedSplash` when your app is done loading resources and you want to navigate to its main page.

```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.Foundation;
using Windows.Foundation.Collections;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Controls.Primitives;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Input;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Navigation;

using Windows.ApplicationModel.Activation;
using SplashScreenExample.Common;
using Windows.UI.Core;

// The Blank Page item template is documented at http://go.microsoft.com/fwlink/p/?LinkID=234238

namespace SplashScreenExample
{
    /// <summary>
    /// An empty page that can be used on its own or navigated to within a Frame.
    /// </summary>
    partial class ExtendedSplash : Page
    {
        internal Rect splashImageRect; // Rect to store splash screen image coordinates.
        private SplashScreen splash; // Variable to hold the splash screen object.
        internal bool dismissed = false; // Variable to track splash screen dismissal status.
        internal Frame rootFrame;

        public ExtendedSplash(SplashScreen splashscreen, bool loadState)
        {
            InitializeComponent();

            // Listen for window resize events to reposition the extended splash screen image accordingly.
            // This is important to ensure that the extended splash screen is formatted properly in response to snapping, unsnapping, rotation, etc...
            Window.Current.SizeChanged += new WindowSizeChangedEventHandler(ExtendedSplash_OnResize);

            splash = splashscreen;

            if (splash != null)
            {
                // Register an event handler to be executed when the splash screen has been dismissed.
                splash.Dismissed += new TypedEventHandler<SplashScreen, Object>(DismissedEventHandler);

                // Retrieve the window coordinates of the splash screen image.
                splashImageRect = splash.ImageLocation;
                PositionImage();

                // Optional: Add a progress ring to your splash screen to show users that content is loading
                PositionRing();
            }

            // Create a Frame to act as the navigation context
            rootFrame = new Frame();

            // Restore the saved session state if necessary
            await RestoreStateAsync(loadState);
        }

        async void RestoreStateAsync(bool loadState)
        {
            if (loadState)
            {
                // TODO: write code to load state
            }
        }

        // Position the extended splash screen image in the same location as the system splash screen image.
        void PositionImage()
        {
            extendedSplashImage.SetValue(Canvas.LeftProperty, splashImageRect.X);
            extendedSplashImage.SetValue(Canvas.TopProperty, splashImageRect.Y);
            extendedSplashImage.Height = splashImageRect.Height;
            extendedSplashImage.Width = splashImageRect.Width;

        }

        void PositionRing()
        {
            splashProgressRing.SetValue(Canvas.LeftProperty, splashImageRect.X + (splashImageRect.Width*0.5) - (splashProgressRing.Width*0.5));
            splashProgressRing.SetValue(Canvas.TopProperty, (splashImageRect.Y + splashImageRect.Height + splashImageRect.Height*0.1));
        }

        void ExtendedSplash_OnResize(Object sender, WindowSizeChangedEventArgs e)
        {
            // Safely update the extended splash screen image coordinates. This function will be fired in response to snapping, unsnapping, rotation, etc...
            if (splash != null)
            {
                // Update the coordinates of the splash screen image.
                splashImageRect = splash.ImageLocation;
                PositionImage();
                PositionRing();
            }
        }

        // Include code to be executed when the system has transitioned from the splash screen to the extended splash screen (application's first view).
        void DismissedEventHandler(SplashScreen sender, object e)
        {
            dismissed = true;

            // Complete app setup operations here...
        }

        void DismissExtendedSplash()
        {
            // Navigate to mainpage
            rootFrame.Navigate(typeof(MainPage));
            // Place the frame in the current Window
            Window.Current.Content = rootFrame;
        }

        void DismissSplashButton_Click(object sender, RoutedEventArgs e)
        {
            DismissExtendedSplash();
        }
    }
}
```

App.xaml.cs: This project was created using the UWP app **Blank App (XAML)** project template in Visual Studio 2015. Both the `OnNavigationFailed` and `OnSuspending` event handlers are automatically generated and don't need to be changed to implement an extended splash screen. This topic only modifies `OnLaunched`.

If you didn't use a project template for your app, see Step 4: [Modify the launch activation handler](#modify-the-launch-activation-handler) for an example of a modified `OnLaunched` that doesn't use [**Frame**](https://msdn.microsoft.com/library/windows/apps/br242682) navigation.

```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.ApplicationModel;
using Windows.ApplicationModel.Activation;
using Windows.Foundation;
using Windows.Foundation.Collections;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Controls.Primitives;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Input;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Navigation;

// The Blank Application template is documented at http://go.microsoft.com/fwlink/p/?LinkID=234227

namespace SplashScreenExample
{
    /// <summary>
    /// Provides application-specific behavior to supplement the default Application class.
    /// </summary>
    sealed partial class App : Application
    {
        /// <summary>
        /// Initializes the singleton application object.  This is the first line of authored code
        /// executed, and as such is the logical equivalent of main() or WinMain().
        /// </summary>
        public App()
        {
            Microsoft.ApplicationInsights.WindowsAppInitializer.InitializeAsync(
            Microsoft.ApplicationInsights.WindowsCollectors.Metadata |
            Microsoft.ApplicationInsights.WindowsCollectors.Session);
            this.InitializeComponent();
            this.Suspending += OnSuspending;
        }

        /// <summary>
        /// Invoked when the application is launched normally by the end user.  Other entry points
        /// will be used such as when the application is launched to open a specific file.
        /// </summary>
        /// <param name="e">Details about the launch request and process.</param>
        protected override void OnLaunched(LaunchActivatedEventArgs e)
        {
#if DEBUG
            if (System.Diagnostics.Debugger.IsAttached)
            {
                this.DebugSettings.EnableFrameRateCounter = true;
            }
#endif

            Frame rootFrame = Window.Current.Content as Frame;

            // Do not repeat app initialization when the Window already has content,
            // just ensure that the window is active
            if (rootFrame == null)
            {
                // Create a Frame to act as the navigation context and navigate to the first page
                rootFrame = new Frame();
                // Set the default language
                rootFrame.Language = Windows.Globalization.ApplicationLanguages.Languages[0];

                rootFrame.NavigationFailed += OnNavigationFailed;

                //  Display an extended splash screen if app was not previously running.
                if (e.PreviousExecutionState != ApplicationExecutionState.Running)
                {
                    bool loadState = (e.PreviousExecutionState == ApplicationExecutionState.Terminated);
                    ExtendedSplash extendedSplash = new ExtendedSplash(e.SplashScreen, loadState);
                    rootFrame.Content = extendedSplash;
                    Window.Current.Content = rootFrame;
                }
            }

            if (rootFrame.Content == null)
            {
                // When the navigation stack isn't restored navigate to the first page,
                // configuring the new page by passing required information as a navigation
                // parameter
                rootFrame.Navigate(typeof(MainPage), e.Arguments);
            }
            // Ensure the current window is active
            Window.Current.Activate();
        }

        /// <summary>
        /// Invoked when Navigation to a certain page fails
        /// </summary>
        /// <param name="sender">The Frame which failed navigation</param>
        /// <param name="e">Details about the navigation failure</param>
        void OnNavigationFailed(object sender, NavigationFailedEventArgs e)
        {
            throw new Exception("Failed to load Page " + e.SourcePageType.FullName);
        }

        /// <summary>
        /// Invoked when application execution is being suspended.  Application state is saved
        /// without knowing whether the application will be terminated or resumed with the contents
        /// of memory still intact.
        /// </summary>
        /// <param name="sender">The source of the suspend request.</param>
        /// <param name="e">Details about the suspend request.</param>
        private void OnSuspending(object sender, SuspendingEventArgs e)
        {
            var deferral = e.SuspendingOperation.GetDeferral();
            // TODO: Save applicaiton state and stop any background activity
            deferral.Complete();
        }
    }
}
```

## Related topics


* [App lifecycle](app-lifecycle.md)

**Reference**

* [**Windows.ApplicationModel.Activation namespace**](https://msdn.microsoft.com/library/windows/apps/br224766)
* [**Windows.ApplicationModel.Activation.SplashScreen class**](https://msdn.microsoft.com/library/windows/apps/br224763)
* [**Windows.ApplicationModel.Activation.SplashScreen.ImageLocation property**](https://msdn.microsoft.com/library/windows/apps/br224765)
* [**Windows.ApplicationModel.Core.CoreApplicationView.Activated event**](https://msdn.microsoft.com/library/windows/apps/br225018)

 

 
