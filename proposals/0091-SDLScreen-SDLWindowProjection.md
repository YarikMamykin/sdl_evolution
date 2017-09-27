# SDLCarWindow Video Projection Developer Interface

```objc
@interface SDLCarWindowConfiguration : NSObject

@property (nonatomic, weak) id<SDLCarWindowDelegate> delegate;
@property (nonatomic, strong) UIViewController *rootViewController; 

@end
```When the property is non-nil, the SDLManager will manage the creation of SDLCarWindow interface instances based on the SDLLifecycleManager state. As it creates SDLCarWindow instances, it will call the app provided delegate, which is defined as follows:

```objc
@class SDLCarWindow;

typedef enum{
    SDLCarWindowStateDisconnected,
    SDLCarWindowStateReadyForStreaming,
    SDLCarWindowStateStreaming,
    SDLCarWindowStateStoppingStreaming
}SDLCarWindowState;

@protocol SDLCarWindowDelegate <NSObject>

- (void)carWindow:(SDLCarWindow *)window didChangeState:(SDLCarWindowState)state;
- (void)carWindow:(SDLCarWindow *)window didEncounterError:(NSError *)error;

@end
```

```objc
@interface SDLCarWindow : NSObject

-(instancetype)init;
-(instancetype)initWithStreamingMediaManager:(SDLStreamingMediaManager *)smm;


-(void)startVideoSessionWithRootViewController:(UIViewController *)vc startBlock:(SDLStreamingStartBlock)sb;

-(void)startTLSVideoSessionWithRootViewController:(UIViewController *)vc (SDLEncryptionFlag)encryptionFlag startBlock:(SDLStreamingEncryptionStartBlock)startBlock;

-(void)stopVideoSession;

@property(nonatomic, assign, readonly) SDLCarWindowState state;
@property(nonatomic, strong, readonly) SDLCarWindowCapabilities *capabilities;
@property(nonatomic, strong) SDLFocusableItemLocator *focusableItemLocator;

@end
```To determine the resolution and aspect ratio of the physical device represented by the SDLCarWindow instance, the app developer will examine the capabilities property of type SDLCarWindowCapabilities, defined as follows:

```objc
typedef enum{
    SDLCarWindowTypeCenterConsole,
    SDLCarWindowTypeInstrumentCluster,
    SDLCarWindowTypeRearSet
}SDLCarWindowType;

@interface SDLCarWindowCapabilities : NSObject

@property(nonatomic, readonly, assign) SDLCarWindowType type;
@property(nonatomic, readonly, assign) CGSize size;
@property(nonatomic, readonly, assign) CGFloat pixelAspectRatio;
@property(nonatomic, readonly, assign) BOOL focusableItemLocatorEnabled;

@end
```

The focusable item selector will be available to the app developer from the SDLCarWindow focusableItemSelector property. This allows the app developer to determine which view has been focused or selected using the viewForTouch SDLHapticHitTester method. 

For head units that allow the app to manage focus and selection, the SDLFocusableItemLocator provides the local logic to do so and interacts with the views through the [UIFocusEnvironment protocol](https://developer.apple.com/documentation/uikit/uifocusenvironment).

### Handling focusable UIButtons
The UIButton class returns NO for the UIFocusItem canBecomeFocused method unless it is being displayed on a CarPlay window. However, the SDLInterfaceManager relies on the canBecomeFocused property to determine which buttons should have spatial data sent to the head unit. To overcome this issue, the SDL proxy will implement the following category on UIButton that will return YES for canBecomeFocused.

```objc
@interface UIButton (SDLFocusable)

@property(nonatomic, readonly) BOOL canBecomeFocused;

@end
```