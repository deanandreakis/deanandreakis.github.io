---
layout: post
title: "Adventures in Swift"
date: 2014-07-12 22:36:50 -0500
comments: true
categories: [apple, Swift]
published: true
---

I was as stunned as everyone when Apple announced a new programming language called Swift at this years World Wide Developers Conference. I had been working on a new iOS app that takes advantage of the Marvel API to display comic book character information and had started the project as usual in Objective-C. I had only just created the basics in the Model layer including the core model objects and relationships along with a wrapper object that encapsulates basic CoreData functionality. I decided that since I was not very far into this new project that it would be a good idea to restart the project in Swift in order to accelerate my learning on the new language. I also decided that the best place to start would be that CoreData wrapper object since it was fairly simple and straightforward. Here is the header and implementation file of the original Objecive-C wrapper object which I call DatabaseManager:

{% highlight swift %}
#import <Foundation/Foundation.h>

@interface DatabaseManager : NSObject

@property (readonly, strong, nonatomic) NSManagedObjectContext *managedObjectContext;
@property (readonly, strong, nonatomic) NSManagedObjectModel *managedObjectModel;
@property (readonly, strong, nonatomic) NSPersistentStoreCoordinator *persistentStoreCoordinator;

+ (DatabaseManager *)sharedDatabaseManager;
- (void) initializeDB;
- (BOOL)isDBPopulated;
- (void)saveContext;
- (NSURL *)applicationDocumentsDirectory;

@end
{% endhighlight %}

And here is it's Objective-C implementation file:

{% highlight swift %}
#import "DatabaseManager.h"
#import "SynthesizeSingleton.h"
#import "DNWAppDelegate.h"
#import "constants.h"

@interface DatabaseManager ()

@property(assign, nonatomic) BOOL isPermObjectsExist;

@end

@implementation DatabaseManager

SYNTHESIZE_SINGLETON_FOR_CLASS(DatabaseManager);

@synthesize managedObjectContext = __managedObjectContext;
@synthesize managedObjectModel = __managedObjectModel;
@synthesize persistentStoreCoordinator = __persistentStoreCoordinator;
@synthesize isPermObjectsExist;

- (void)saveContext
{
    NSError *error = nil;
    NSManagedObjectContext *managedObjectContext = self.managedObjectContext;
    if (managedObjectContext != nil)
    {
        if ([managedObjectContext hasChanges] && ![managedObjectContext save:&error])
        {
            //NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
            //abort();
        } 
    }
}

#pragma mark - Core Data stack

/**
 Returns the managed object context for the application.
 If the context doesn't already exist, it is created and bound to the persistent store coordinator for the application.
 */
- (NSManagedObjectContext *)managedObjectContext
{
    if (__managedObjectContext != nil)
    {
        return __managedObjectContext;
    }
    
    NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];
    if (coordinator != nil)
    {
        __managedObjectContext = [[NSManagedObjectContext alloc] init];
        [__managedObjectContext setPersistentStoreCoordinator:coordinator];
    }
    return __managedObjectContext;
}

/**
 Returns the managed object model for the application.
 If the model doesn't already exist, it is created from the application's model.
 */
- (NSManagedObjectModel *)managedObjectModel
{
    if (__managedObjectModel != nil)
    {
        return __managedObjectModel;
    }
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"MarvelMania" withExtension:@"momd"];
    __managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    return __managedObjectModel;
}

/**
 Returns the persistent store coordinator for the application.
 If the coordinator doesn't already exist, it is created and the application's store added to it.
 */
- (NSPersistentStoreCoordinator *)persistentStoreCoordinator
{
    if (__persistentStoreCoordinator != nil)
    {
        return __persistentStoreCoordinator;
    }
    
    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"MarvelMania.sqlite"];
    
    NSError *error = nil;
    __persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:[self managedObjectModel]];
    if (![__persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error])
    {
        
        //NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
        //abort();
    }    
    
    return __persistentStoreCoordinator;
}

#pragma mark - Application's Documents directory

/**
 Returns the URL to the application's Documents directory.
 */
- (NSURL *)applicationDocumentsDirectory
{
    return [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
}

- (void)initializeDB
{
    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"MarvelMania.sqlite"];
    [[NSFileManager defaultManager] removeItemAtURL:storeURL error:nil];
}


 - (BOOL)isDBPopulated
 {
     NSError *error;
     NSManagedObjectContext *context = [self managedObjectContext];
     NSFetchRequest *fetchRequest = [[NSFetchRequest alloc] init];
     NSEntityDescription *entity = [NSEntityDescription entityForName:@"Character"
                                               inManagedObjectContext:context];
     [fetchRequest setEntity:entity];
     NSArray *fetchedObjects = [context executeFetchRequest:fetchRequest error:&error];
     if([fetchedObjects count] > 0)
     {
         return TRUE;
     }
     else
     {
         return FALSE;
     }
 }

@end
{% endhighlight %}

Here is the same implementation is Swift:

{% highlight swift %}
import Foundation
import CoreData

class DatabaseManager {
    
    var managedObjectContext : NSManagedObjectContext? {
    
    get {
        if (self.managedObjectContext != nil)
        {
            return self.managedObjectContext
        }
        
        var coordinator = self.persistentStoreCoordinator?
        var moc : NSManagedObjectContext = NSManagedObjectContext()
        if (coordinator != nil)
        {
            moc.persistentStoreCoordinator = coordinator
        }
        return moc
    }
    }
    
    var managedObjectModel : NSManagedObjectModel? {
    
    get {
        if (self.managedObjectModel != nil)
        {
            return self.managedObjectModel;
        }
        
        let modelURL : NSURL = NSBundle.mainBundle().URLForResource("MarvelCharacters", withExtension: "momd")
        var mom : NSManagedObjectModel = NSManagedObjectModel(contentsOfURL: modelURL)
        return mom
    }
    }
    
    var persistentStoreCoordinator : NSPersistentStoreCoordinator? {
    
    get {
        if (self.persistentStoreCoordinator != nil)
        {
            return self.persistentStoreCoordinator;
        }
        
        var storeURL : NSURL?
        storeURL = applicationDocumentsDirectory()?.URLByAppendingPathComponent("MarvelMania.sqlite")?
        
        var error : NSError?
        var psc : NSPersistentStoreCoordinator = NSPersistentStoreCoordinator(managedObjectModel: self.managedObjectModel)
        
        if(!(psc.addPersistentStoreWithType(NSSQLiteStoreType, configuration:nil, URL: storeURL, options: nil, error: &error)))
        {
            
        }
        
        return psc
    }
    }
    
    class func shared()->DatabaseManager {
        struct Static {
            static let s_manager = DatabaseManager()
        }
        return Static.s_manager
    }
    
    func initializeDB() {
        var storeURL : NSURL?
        var saveError : NSError?
        
        storeURL = applicationDocumentsDirectory()?.URLByAppendingPathComponent("MarvelCharacters.sqlite")?
        NSFileManager.defaultManager().removeItemAtURL(storeURL, error: &saveError)
    }
    
    func isDBPopulated()->Bool {
        return true
    }
    
    func saveContext() {
        var saveError : NSError?
        
        if managedObjectContext?.hasChanges {
            let written = managedObjectContext?.save(&saveError)
            if !written {
                // place to put error handling code
                if let error = saveError {
                  println("write failure: \(error.localizedDescription)")
                }
            }
        }
    }
    
    func applicationDocumentsDirectory()->NSURL? {
        return NSFileManager.defaultManager()?.URLsForDirectory(NSSearchPathDirectory.DocumentDirectory, inDomains: NSSearchPathDomainMask.UserDomainMask)?.bridgeToObjectiveC().lastObject as? NSURL
    }
}
{% endhighlight %}

Here are some initial observations having ported this DatabaseManager object:

1. No headers in Swift; all definition and implementation are in a single .swift file.
2. Optionals in Swift: Objects that may return nil are marked with a question mark (?).
3. Syntax differences: methods of an object are defined with the "func" keyword and return types use the "->" syntax.
4. DatabaseManager has three (3) properties all with custom getters. Since these properties will not change once initialized I could probably declare them with the "let" keyword instead which declares them as essentially immutable.
5. If you buy into the [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter) principle then Swift is not for you!

This object still has some work needed (e.g. the isDBPopulated method still needs an implementation and error handling throughout still needs some thought.). I plan on writing unit test cases for it as well.


My plan is to continue to make posts on my experience writing my first application in Swift. Stay tuned!
