
[Source](http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial "Permalink to AFNetworking 2.0 Tutorial - Ray Wenderlich")

# AFNetworking 2.0 Tutorial - Ray Wenderlich

In iOS 7, Apple introduced NSURLSession as the new, preferred method of networking (as opposed to the older NSURLConnection API). Using this raw NSURLSession API is definitely a valid way to write your networking code – we even have a [tutorial on that][4].

iOS 7 引入了新的 NSURLSession, 比旧的 NSURLConnection API 好用。

However, there's an alternative to consider – using the popular third party networking library [AFNetworking][5]. The latest version of AFNetworking (2.0) is now built on top of NSURLSession, so you get all of the great features provided there. But you also get a lot of extra cool features – like serialization, reachability support, UIKit integration (such as a handy category on asynchronously loading images in a UIImageView), and more.

另一个选择是非常流行的第三方网络库 AFNetworking. 最新版的 AFNetworking 就是基于 NSURLSession 的。

AFNetworking is incredibly popular – it won our Reader's Choice [2012 Best iOS Library Award][6]. It's also one of the most widely used, open-source projects with over 10,000 stars, 2,600 forks, and 160 contributors on [Github][7].

In this AFNetworking 2.0 tutorial, you will learn about the major components of AFNetworking by building a Weather App that uses feeds from [World Weather Online][8]. You'll start with static weather data, but by the end of the tutorial, the app will be fully connected to live weather feeds.

Today's forecast: a cool developer learns all about AFNetworking and gets inspired to use it in his/her apps. Let's get busy!

## Getting Started

First download the starter project for this AFNetworking 2.0 tutorial [here][9].

This project provides a basic UI to get you started – no AFNetworking code has been added yet.

Open _MainStoryboard.storyboard_, and you will see 3 view controllers:
  
![Weather App Overview][10]  

From left to right, they are:

* A top-level navigation controller
* A table view controller that will display the weather, one row per day
* A custom view controller (WeatherAnimationViewController) that will show the weather for a single day when the user taps on a table view cell

Build and run the project. You'll see the UI appear, but nothing works yet. That's because the app needs to get its data from the network, but this code hasn't been added yet. This is what you will be doing in this tutorial!

下载并解压 AFNetworking, 把其中的 *AFNetworking* 和 *UIKit+AFNetworking* 两个子目录拖拽到自己的 Xcode project.

![Include AFNetworking][13]

When presented with options for adding the folders, make sure that _Copy items into destination group's folder (if needed)_ and _Create groups for any added folders_ are both checked.

打开 *Weather-Prefix.pch* 预编译头文件，加入以下一行：

`#import "AFNetworking.h"`

Adding AFNetworking to the pre-compiled header means that the framework will be automatically included in all the project's source files.

Pretty easy, eh? Now you're ready to "weather" the code!

## Operation JSON

AFNetworking 既支持传统的 HTTP 请求，也支持 JSON, XML, plist 等结构化数据——虽然内置的 `NSJSONSerialization` 也可以解析 JSON.

First you need the base URL of the test script. Add this to the top of _WTTableViewController.m_, just underneath all the #import lines.

    static [NSString][15] * const BaseURLString = @"http://www.raywenderlich.com/demos/weather_sample/";

This is the URL to an incredibly simple "web service" that I created for you for this tutorial. If you're curious what it looks like, you can [download the source][16].

The web service returns weather data in 3 different formats – JSON, XML, and PLIST. You can take a look at the data it can return by using these URLS:

- http://www.raywenderlich.com/demos/weather_sample/weather.php?format=json
- http://www.raywenderlich.com/demos/weather_sample/weather.php?format=xml
- http://www.raywenderlich.com/demos/weather_sample/weather.php?format=plist (might not show correctly in your browser)

The first data format you will be using is [JSON][17]. JSON is a very common JavaScript-derived object format. It looks something like this:

    {
        "data": {
            "current_condition": [
                {
                    "cloudcover": "16",
                    "humidity": "59",
                    "observation_time": "09:09 PM",
                }
            ]
        }
    }

When the user taps the JSON button, the app will load and process JSON data from the server. In _WTTableViewController.m_, find the _jsonTapped:_ method (it should be empty) and replace it with the following:

```
- (IBAction)jsonTapped:(id)sender
{
    // 1
    NSString *string = [NSString stringWithFormat:@"%@weather.php?format=json", BaseURLString];
    NSURL *url = [NSURL URLWithString:string];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
 
    // 2
    AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
    operation.responseSerializer = [AFJSONResponseSerializer serializer];
 
    [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
 
        // 3
        self.weather = (NSDictionary *)responseObject;
        self.title = @"JSON Retrieved";
        [self.tableView reloadData];
 
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
 
        // 4
        UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Error Retrieving Weather"
                                                            message:[error localizedDescription]
                                                           delegate:nil
                                                  cancelButtonTitle:@"Ok"
                                                  otherButtonTitles:nil];
        [alertView show];
    }];
 
    // 5
    [operation start];
}

```

Awesome, this is your first AFNetworking code! Since this is all new, I'll explain it one section at a time.

1. You first create a string representing the full url from the base URL string. This is then used to create an NSURL object, which is used to make an NSURLRequest.
2. AFHTTPRequestOperation is an all-in-one class for handling HTTP transfers across the network. You tell it that the response should be read as JSON by setting the _responseSerializer_ property to the default JSON serializer. AFNetworking will then take care of parsing the JSON for you.
3. The success block runs when (surprise!) the request succeeds. The JSON serializer parses the received data and returns a dictionary in the responseObject variable, which is stored in the _weather_ property.
4. The failure block runs if something goes wrong – such as if networking isn't available. If this happens, you simply display an alert with the error message.
5. You must explicitly tell the operation to "start" (or else nothing will happen).

As you can see, AFNetworking is extremely simple to use. In just a few lines of code, you were able to create a networking operation that both downloads and parses its response.

Now that the weather data is stored in _self.weather_, you need to display it. Find the _tableView:numberOfRowsInSection:_ method and replace it with the following:

| ----- |
|

    - (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
    {
        if(!self.weather)
            return 0;
    &nbsp;
        switch (section) {
            case 0: {
                return 1;
            }
            case 1: {
                [NSArray][22] *upcomingWeather = [self.weather upcomingWeather];
                return [upcomingWeather count];
            }
            default:
                return 0;
        }
    }

 |

The table view will have two sections: the first to display the current weather and the second to display the upcoming weather.

"Wait a minute!", you might be thinking. What is this [_self.weather upcomingWeather]_? If self.weather is a plain old NSDictionary, how does it know what "upcomingWeather" is?

To make it easier to display the data, I added a couple of helper categories on NSDictionary in the starter project:

* NSDictionary+weather
* NSDictionary+weather_package

These categories add some handy methods that make it a little easier to access the data elements. You want to focus on the networking part and not on navigating NSDictionary keys, right?

_Note:_ FYI, an alternative way to make working with JSON results a bit easier than looking up keys in dictionaries or creating special categories like this is to use a third party library like [JSONModel][23].

Still in _WTTableViewController.m_, find the _tableView:cellForRowAtIndexPath:_ method and replace it with the following implementation:

| ----- |
|

    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:([NSIndexPath][24] *)indexPath
    {
        static [NSString][15] *CellIdentifier = @"WeatherCell";
        UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier forIndexPath:indexPath];
    &nbsp;
        [NSDictionary][20] *daysWeather = nil;
    &nbsp;
        switch (indexPath.section) {
            case 0: {
                daysWeather = [self.weather currentCondition];
                break;
            }
    &nbsp;
            case 1: {
                [NSArray][22] *upcomingWeather = [self.weather upcomingWeather];
                daysWeather = upcomingWeather[indexPath.row];
                break;
            }
    &nbsp;
            default:
                break;
        }
    &nbsp;
        cell.textLabel.text = [daysWeather weatherDescription];
    &nbsp;
        // You will add code here later to customize the cell, but it's good for now.
    &nbsp;
        return cell;
    }

 |

Like the tableView:numberOfRowsInSection: method, the handy NSDictionary categories are used to easily access the data. The current day's weather is a dictionary, and the upcoming days are stored in an array.

Build and run your project; tap on the _JSON_ button to get the networking request in motion; and you should see this:

  
![JSON Retrieved][25]  

JSON success!

## Operation Property Lists

Property lists (or plists for short) are just XML files structured in a certain way (defined by Apple). Apple uses them all over the place for things like storing user settings. They look something like this:

| ----- |
|

    &nbsp;
    <dict>
      <key>data</key>
      <dict>
        <key>current_condition</key>
          <array>
          <dict>
            <key>cloudcover</key>
            <string>16</string>
            <key>humidity</key>
            <string>59</string>
            ...

 |

The above represents:

* A dictionary with a single key called "data" that contains another dictionary.
* That dictionary has a single key called "current_condition" that contains an array.
* That array contains a dictionary with several keys and values, like cloudcover=16 and humidity=59.

It's time to load the plist version of the weather data. Find the _plistTapped:_ method and replace the empty implementation with the following:

| ----- |
|

    - (IBAction)plistTapped:(id)sender
    {
        [NSString][15] *string = [[NSString][15] stringWithFormat:@"%@weather.php?format=plist", BaseURLString];
        [NSURL][18] *url = [[NSURL][18] URLWithString:string];
        [NSURLRequest][19] *request = [[NSURLRequest][19] requestWithURL:url];
    &nbsp;
        AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
    &nbsp;
        // Make sure to set the responseSerializer correctly
        operation.responseSerializer = [AFPropertyListResponseSerializer serializer];
    &nbsp;
        [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
    &nbsp;
            self.weather = ([NSDictionary][20] *)responseObject;
            self.title = @"PLIST Retrieved";
            [self.tableView reloadData];
    &nbsp;
        } failure:^(AFHTTPRequestOperation *operation, [NSError][21] *error) {
    &nbsp;
            UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Error Retrieving Weather"
                                                                message:[error localizedDescription]
                                                               delegate:nil
                                                      cancelButtonTitle:@"Ok"
                                                      otherButtonTitles:nil];
            [alertView show];
        }];
    &nbsp;
        [operation start];
    }

 |

Notice that this code is almost identical to the JSON version, except for changing the _responseSerializer_ to the default _AFPropertyListResponseSerializer_ to let AFNetworking know that you're going to be parsing a plist.

That's pretty neat: your app can accept either JSON or plist formats with just a tiny change to the code!

Build and run your project and try tapping on the _PLIST_ button. You should see something like this:

  
![PLIST Retrieved][26]  

The _Clear_ button in the top navigation bar will clear the title and table view data so you can reset everything to make sure the requests are going through.

## Operation XML

While AFNetworking handles JSON and plist parsing for you, working with XML is a little more complicated. This time, it's your job to construct the weather dictionary from the XML feed.

Fortunately, iOS provides some help via the NSXMLParser class (which is a [SAX parser][27], if you want to read up on it).

Still in _WTTableViewController.m_, find the _xmlTapped:_ method and replace its implementation with the following:

| ----- |
|

    - (IBAction)xmlTapped:(id)sender
    {
        [NSString][15] *string = [[NSString][15] stringWithFormat:@"%@weather.php?format=xml", BaseURLString];
        [NSURL][18] *url = [[NSURL][18] URLWithString:string];
        [NSURLRequest][19] *request = [[NSURLRequest][19] requestWithURL:url];
    &nbsp;
        AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
    &nbsp;
        // Make sure to set the responseSerializer correctly
        operation.responseSerializer = [AFXMLParserResponseSerializer serializer];
    &nbsp;
        [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
    &nbsp;
            [NSXMLParser][28] *XMLParser = ([NSXMLParser][28] *)responseObject;
            [XMLParser setShouldProcessNamespaces:YES];
    &nbsp;
            // Leave these commented for now (you first need to add the delegate methods)
            // XMLParser.delegate = self;
            // [XMLParser parse];
    &nbsp;
        } failure:^(AFHTTPRequestOperation *operation, [NSError][21] *error) {
    &nbsp;
            UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Error Retrieving Weather"
                                                                message:[error localizedDescription]
                                                               delegate:nil
                                                      cancelButtonTitle:@"Ok"
                                                      otherButtonTitles:nil];
            [alertView show];
    &nbsp;
        }];
    &nbsp;
        [operation start];
    }

 |

This should look pretty familiar by now. The biggest change is that in the success block you don't get a nice, preprocessed NSDictionary object passed to you. Instead, responseObject is an instance of NSXMLParser, which you will use to do the heavy lifting in parsing the XML.

You'll need to implement a set of delegate methods for NXMLParser to be able to parse the XML. Notice that XMLParser's delegate is set to _self_, so you will need to add NSXMLParser's delegate methods to WTTableViewController to handle the parsing.

First, update _WTTableViewController.h_ and change the class declaration at the top as follows:

| ----- |
|

    @interface WTTableViewController : UITableViewController<nsxmlparserdelegate>

 |

This means the class will implement the NSXMLParserDelegate protocol. You will implement these methods soon, but first you need to add a few properties.

Add the following properties to _WTTableViewController.m_ within the class extension, right after _@interface WTTableViewController ()_ :

These properties will come in handy when you're parsing the XML.

Now paste this method in _WTTableViewController.m_, right before _@end_:

The parser calls this method when it first starts parsing. When this happens, you set _self.xmlWeather_ to a new dictionary, which will hold hold the XML data.

Next paste this method right after this previous one:

| ----- |
|

    - (void)parser:([NSXMLParser][28] *)parser didStartElement:([NSString][15] *)elementName namespaceURI:([NSString][15] *)namespaceURI qualifiedName:([NSString][15] *)qName attributes:([NSDictionary][20] *)attributeDict
    {
        self.elementName = qName;
    &nbsp;
        if([qName isEqualToString:@"current_condition"] ||
           [qName isEqualToString:@"weather"] ||
           [qName isEqualToString:@"request"]) {
            self.currentDictionary = [[NSMutableDictionary][29] dictionary];
        }
    &nbsp;
        self.outstring = [[NSMutableString][30] string];
    }

 |

The parser calls this method when it finds a new element start tag. When this happens, you keep track of the new element's name as _self.elementName_ and then set _self.currentDictionary_ to a new dictionary if the element name represents the start of a new weather forecast. You also reset _outstring_ as a new mutable string in preparation for new XML to be received related to the element.

Next paste this method just after the previous one:

| ----- |
|

    - (void)parser:([NSXMLParser][28] *)parser foundCharacters:([NSString][15] *)string
    {
        if (!self.elementName)
            return;
    &nbsp;
        [self.outstring appendFormat:@"%@", string];
    }

 |

As the name suggests, the parser calls this method when it finds new characters on an XML element. You append the new characters to outstring, so they can be processed once the XML tag is closed.

Again, paste this next method just after the previous one:

| ----- |
|

    - (void)parser:([NSXMLParser][28] *)parser didEndElement:([NSString][15] *)elementName namespaceURI:([NSString][15] *)namespaceURI qualifiedName:([NSString][15] *)qName
    {
        // 1
        if ([qName isEqualToString:@"current_condition"] ||
           [qName isEqualToString:@"request"]) {
            self.xmlWeather[qName] = @[self.currentDictionary];
            self.currentDictionary = nil;
        }
        // 2
        else if ([qName isEqualToString:@"weather"]) {
    &nbsp;
            // Initialize the list of weather items if it doesn't exist
            [NSMutableArray][31] *array = self.xmlWeather[@"weather"] ?: [[NSMutableArray][31] array];
    &nbsp;
            // Add the current weather object
            [array addObject:self.currentDictionary];
    &nbsp;
            // Set the new array to the "weather" key on xmlWeather dictionary
            self.xmlWeather[@"weather"] = array;
    &nbsp;
            self.currentDictionary = nil;
        }
        // 3
        else if ([qName isEqualToString:@"value"]) {
            // Ignore value tags, they only appear in the two conditions below
        }
        // 4
        else if ([qName isEqualToString:@"weatherDesc"] ||
                [qName isEqualToString:@"weatherIconUrl"]) {
            [NSDictionary][20] *dictionary = @{@"value": self.outstring};
            [NSArray][22] *array = @[dictionary];
            self.currentDictionary[qName] = array;
        }
        // 5
        else if (qName) {
            self.currentDictionary[qName] = self.outstring;
        }
    &nbsp;
    	self.elementName = nil;
    }

 |

This method is called when an end element tag is encountered. When that happens, you check for a few special tags:

1. The _current_condition_ element indicates you have the weather for the current day. You add this directly to the xmlWeather dictionary.
2. The _weather_ element means you have the weather for a subsequent day. While there is only one current day, there may be several subsequent days, so you add this weather information to an array.
3. The _value_ tag only appears inside other tags, so it's safe to skip over it.
4. The _weatherDesc_ and _weatherIconUrl_ element values need to be boxed inside an array before they can be stored. This way, they will match how the JSON and plist versions of the data are structured exactly.
5. All other elements can be stored as is.

Now for the final delegate method! Paste this method just after the previous one:

| ----- |
|

    - (void) parserDidEndDocument:([NSXMLParser][28] *)parser
    {
        self.weather = @{@"data": self.xmlWeather};
        self.title = @"XML Retrieved";
        [self.tableView reloadData];
    }

 |

The parser calls this method when it reaches the end of the document. At this point, the xmlWeather dictionary that you've been building is complete, so the table view can be reloaded.

Wrapping xmlWeather inside another NSDictionary might seem redundant, but this ensures the format matches up exactly with the JSON and plist versions. This way, all three data formats can be displayed with the same code!

  
![][32]  

Now that the delegate methods and properties are in place, return to the _xmlTapped:_ method and uncomment the lines of code from before:

| ----- |
|

    - (IBAction)xmlTapped:(id)sender
    {
        ...
        [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
    &nbsp;
            [NSXMLParser][28] *XMLParser = ([NSXMLParser][28] *)responseObject;
            [XMLParser setShouldProcessNamespaces:YES];
    &nbsp;
            // These lines below were previously commented
            XMLParser.delegate = self;
            [XMLParser parse];
        ...
    }

 |

Build and run your project. Try tapping the XML button, and you should see this:

  
![XML Retrieved][33]  

## A Little Weather Flair

Hmm, that looks dreary, like a week's worth of rainy days. How could you jazz up the weather information in your table view?

Take another peak at the [JSON format from before][34], and you will see that there are image URLs for each weather item. Displaying these weather images in each table view cell would add some visual interest to the app.

AFNetworking adds a category to UIImageView that lets you load images asynchronously, meaning the UI will remain responsive while images are downloaded in the background. To take advantage of this, first add the category import to the top of _WTTableViewController.m_:

| ----- |
|

    #import "UIImageView+AFNetworking.h"

 |

Find the _tableView:cellForRowAtIndexPath:_ method and paste the following code just above the final _return cell;_ line (there should be a comment marking the spot):

| ----- |
|

     cell.textLabel.text = [daysWeather weatherDescription];
    &nbsp;
        [NSURL][18] *url = [[NSURL][18] URLWithString:daysWeather.weatherIconURL];
        [NSURLRequest][19] *request = [[NSURLRequest][19] requestWithURL:url];
        UIImage *placeholderImage = [UIImage imageNamed:@"placeholder"];
    &nbsp;
        __weak UITableViewCell *weakCell = cell;
    &nbsp;
        [cell.imageView setImageWithURLRequest:request
                              placeholderImage:placeholderImage
                                       success:^([NSURLRequest][19] *request, [NSHTTPURLResponse][35] *response, UIImage *image) {
    &nbsp;
                                           weakCell.imageView.image = image;
                                           [weakCell setNeedsLayout];
    &nbsp;
                                       } failure:nil];

 |

UIImageView+AFNetworking makes _setImageWithURLRequest:_ and several other related methods available to you.

Both the success and failure blocks are optional, but if you do provide a success block, you must explicitly set the image property on the image view (or else it won't be set). If you don't provide a success block, the image will automatically be set for you.

When the cell is first created, its image view will display the placeholder image until the real image has finished downloading.

Now build and run your project. Tap on any of the operations you've added so far, and you should see this:

  
![UIImageView-AFNetworking Example][36]  

Nice! Asynchronously loading images has never been easier.

## A RESTful Class

So far you've been creating one-off networking operations using AFHTTPRequestOperation.

Alternatively, _AFHTTPRequestOperationManager_ and _AFHTTPSessionManager_ are designed to help you easily interact with a single, web-service endpoint.

Both of these allow you to set a base URL and then make several requests to the same endpoint. Both can also monitor for changes in connectivity, encode parameters, handle multipart form requests, enqueue batch operations, and help you perform the full suite of RESTful verbs (GET, POST, PUT, and DELETE).

"Which one should I use?", you might ask.

* _If you're targeting iOS 7 and above_, use AFHTTPSessionManager, as internally it creates and uses NSURLSession and related objects.
* _If you're targeting iOS 6 and above_, use AFHTTPRequestOperationManager, which has similar functionality to AFHTTPSessionManager, yet it uses NSURLConnection internally instead of NSURLSession (which isn't available in iOS 6). Otherwise, these classes are very similar in functionality.

In your weather app project, you'll be using AFHTTPSessionManager to perform both a GET and PUT operation.

_Note:_ Unclear on what all this talk is about REST, GET, and POST? Check out this explanation of the subject – [What is REST?][37]

Update the class declaration at the top of _WTTableViewController.h_ to the following:

| ----- |
|

    @interface WTTableViewController : UITableViewController<nsxmlparserdelegate, cllocationmanagerdelegate,="" uiactionsheetdelegate="">

 |

In _WTTableViewController.m_, find the _clientTapped:_ method and replace its implementation with the following:

| ----- |
|

    - (IBAction)clientTapped:(id)sender
    {
        UIActionSheet *actionSheet = [[UIActionSheet alloc] initWithTitle:@"AFHTTPSessionManager"
                                                                 delegate:self
                                                        cancelButtonTitle:@"Cancel"
                                                   destructiveButtonTitle:nil
                                                        otherButtonTitles:@"HTTP GET", @"HTTP POST", nil];
        [actionSheet showFromBarButtonItem:sender animated:YES];
    }

 |

This method creates and displays an action sheet asking the user to choose between a GET and POST request. Add the following method at the end of the class implementation (right before _@end_) to implement the action sheet delegate method:

| ----- |
|

    - (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex
    {
        if (buttonIndex == [actionSheet cancelButtonIndex]) {
            // User pressed cancel -- abort
            return;
        }
    &nbsp;
        // 1
        [NSURL][18] *baseURL = [[NSURL][18] URLWithString:BaseURLString];
        [NSDictionary][20] *parameters = @{@"format": @"json"};
    &nbsp;
        // 2
        AFHTTPSessionManager *manager = [[AFHTTPSessionManager alloc] initWithBaseURL:baseURL];
        manager.responseSerializer = [AFJSONResponseSerializer serializer];
    &nbsp;
        // 3
        if (buttonIndex == 0) {
            [manager GET:@"weather.php" parameters:parameters success:^(NSURLSessionDataTask *task, id responseObject) {
                self.weather = responseObject;
                self.title = @"HTTP GET";
                [self.tableView reloadData];
            } failure:^(NSURLSessionDataTask *task, [NSError][21] *error) {
                UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Error Retrieving Weather"
                                                                    message:[error localizedDescription]
                                                                   delegate:nil
                                                          cancelButtonTitle:@"Ok"
                                                          otherButtonTitles:nil];
                [alertView show];
            }];
        }
    &nbsp;
        // 4
        else if (buttonIndex == 1) {
            [manager POST:@"weather.php" parameters:parameters success:^(NSURLSessionDataTask *task, id responseObject) {
                self.weather = responseObject;
                self.title = @"HTTP POST";
                [self.tableView reloadData];
            } failure:^(NSURLSessionDataTask *task, [NSError][21] *error) {
                UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Error Retrieving Weather"
                                                                    message:[error localizedDescription]
                                                                   delegate:nil
                                                          cancelButtonTitle:@"Ok"
                                                          otherButtonTitles:nil];
                [alertView show];
            }];
        }
    }

 |

Here's what's happening above:

1. You first set up the baseURL and the dictionary of parameters.
2. You then create an instance of AFHTTPSessionManager and set its responseSerializer to the default JSON serializer, similar to the previous JSON example.
3. If the user presses the button index for HTTP GET, you call the GET method on the manager, passing in the parameters and usual pair of success and failure blocks.
4. You do the same with the POST version.

In this example you're requesting JSON responses, but you can easily request either of the other two formats as discussed previously.

Build and run your project, tap on the _Client_ button and then tap on either the _HTTP GET_ or _HTTP POST_ button to initiate the associated request. You should see these screens:

  
![AFHTTPSessionManager GET and POST][38]  

At this point, you know the basics of using AFHTTPSessionManager, but there's an even better way to use it that will result in cleaner code, which you'll learn about next.

## World Weather Online

Before you can use the live service, you'll first need to register for a free account on [World Weather Online][39]. Don't worry – it's quick and easy to do!

After you've registered, you should receive a confirmation email at the address you provided, which will have a link to confirm your email address (required). You then need to request a free API key via the _My Account_ page. Go ahead and leave the page open with your API key as you'll need it soon.

Now that you've got your API key, back to AFNetworking…

## Hooking into the Live Service

So far you've been creating AFHTTPRequestOperation and AFHTTPSessionManager directly from the table view controller as you needed them. More often than not, your networking requests will be associated with a single web service or API.

AFHTTPSessionManager has everything you need to talk to a web API. It will decouple your networking communications code from the rest of your code, and make your networking communications code reusable throughout your project.

Here are two guidelines on AFHTTPSessionManager best practices:

1. Create a subclass for each web service. For example, if you're writing a social network aggregator, you might want one subclass for Twitter, one for Facebook, another for Instragram and so on.
2. In each AFHTTPSessionManager subclass, create a class method that returns a shared singleton instance. This saves resources and eliminates the need to allocate and spin up new objects.

Your project currently doesn't have a subclass of AFHTTPSessionManager; it just creates one directly. Let's fix that.

To begin, create a new file in your project of type _iOSCocoa TouchObjective-C Class_. Call it _WeatherHTTPClient_ and make it a subclass of _AFHTTPSessionManager_.

You want the class to do three things: perform HTTP requests, call back to a delegate when the new weather data is available, and use the user's physical location to get accurate weather.

Replace the contents of _WeatherHTTPClient.h_ with the following:

| ----- |
|

    #import "AFHTTPSessionManager.h"
    &nbsp;
    @protocol WeatherHTTPClientDelegate;
    &nbsp;
    @interface WeatherHTTPClient : AFHTTPSessionManager
    @property (nonatomic, weak) id<weatherhttpclientdelegate>delegate;
    &nbsp;
    + (WeatherHTTPClient *)sharedWeatherHTTPClient;
    - (instancetype)initWithBaseURL:([NSURL][18] *)url;
    - (void)updateWeatherAtLocation:(CLLocation *)location forNumberOfDays:(NSUInteger)number;
    &nbsp;
    @end
    &nbsp;
    @protocol WeatherHTTPClientDelegate <nsobject>
    @optional
    -(void)weatherHTTPClient:(WeatherHTTPClient *)client didUpdateWithWeather:(id)weather;
    -(void)weatherHTTPClient:(WeatherHTTPClient *)client didFailWithError:([NSError][21] *)error;
    @end

 |

You'll learn more about each of these methods as you implement them. Switch over to _WeatherHTTPClient.m_ and add the following right after the import statement:

| ----- |
|

    // Set this to your World Weather Online API Key
    static [NSString][15] * const WorldWeatherOnlineAPIKey = @"PASTE YOUR API KEY HERE";
    &nbsp;
    static [NSString][15] * const WorldWeatherOnlineURLString = @"http://api.worldweatheronline.com/free/v1/";

 |

Make sure you replace _@"PASTE YOUR KEY HERE"_ with your actual World Weather Online API Key.

Next paste these methods just after the @implementation line:

| ----- |
|

    + (WeatherHTTPClient *)sharedWeatherHTTPClient
    {
        static WeatherHTTPClient *_sharedWeatherHTTPClient = nil;
    &nbsp;
        static dispatch_once_t onceToken;
        dispatch_once(&amp;onceToken, ^{
            _sharedWeatherHTTPClient = [[self alloc] initWithBaseURL:[[NSURL][18] URLWithString:WorldWeatherOnlineURLString]];
        });
    &nbsp;
        return _sharedWeatherHTTPClient;
    }
    &nbsp;
    - (instancetype)initWithBaseURL:([NSURL][18] *)url
    {
        self = [super initWithBaseURL:url];
    &nbsp;
        if (self) {
            self.responseSerializer = [AFJSONResponseSerializer serializer];
            self.requestSerializer = [AFJSONRequestSerializer serializer];
        }
    &nbsp;
        return self;
    }

 |

The _sharedWeatherHTTPClient_ method uses Grand Central Dispatch to ensure the shared singleton object is only allocated once. You initialize the object with a base URL and set it up to request and expect JSON responses from the web service.

Paste the following method underneath the previous ones:

| ----- |
|

    - (void)updateWeatherAtLocation:(CLLocation *)location forNumberOfDays:(NSUInteger)number
    {
        [NSMutableDictionary][29] *parameters = [[NSMutableDictionary][29] dictionary];
    &nbsp;
        parameters[@"num_of_days"] = @(number);
        parameters[@"q"] = [[NSString][15] stringWithFormat:@"%f,%f",location.coordinate.latitude,location.coordinate.longitude];
        parameters[@"format"] = @"json";
        parameters[@"key"] = WorldWeatherOnlineAPIKey;
    &nbsp;
        [self GET:@"weather.ashx" parameters:parameters success:^(NSURLSessionDataTask *task, id responseObject) {
            if ([self.delegate respondsToSelector:@selector(weatherHTTPClient:didUpdateWithWeather:)]) {
                [self.delegate weatherHTTPClient:self didUpdateWithWeather:responseObject];
            }
        } failure:^(NSURLSessionDataTask *task, [NSError][21] *error) {
            if ([self.delegate respondsToSelector:@selector(weatherHTTPClient:didFailWithError:)]) {
                [self.delegate weatherHTTPClient:self didFailWithError:error];
            }
        }];
    }

 |

This method calls out to World Weather Online to get the weather for a particular location.

Once the object has loaded the weather data, it needs some way to communicate that data back to whoever's interested. Thanks to the WeatherHTTPClientDelegate protocol and its delegate methods, the success and failure blocks in the above code can notify a controller that the weather has been updated for a given location. That way, the controller can update what it is displaying.

Now it's time to put the final pieces together! The WeatherHTTPClient is expecting a location and has a defined delegate protocol, so you need to update the WTTableViewController class to take advantage of this.

Open up _WTTableViewController.h_ to add an import and replace the @interface declaration as follows:

| ----- |
|

    #import "WeatherHTTPClient.h"
    &nbsp;
    @interface WTTableViewController : UITableViewController <nsxmlparserdelegate, cllocationmanagerdelegate,="" uiactionsheetdelegate,="" weatherhttpclientdelegate="">

 |

Also add a new Core Location manager property:

| ----- |
|

    @property (nonatomic, strong) CLLocationManager *locationManager;

 |

In _WTTableViewController.m_, add the following lines to the bottom of _viewDidLoad:_:

| ----- |
|

    self.locationManager = [[CLLocationManager alloc] init];
    self.locationManager.delegate = self;

 |

These lines initialize the Core Location manager to determine the user's location when the view loads. The Core Location manager then reports that location via a delegate callback. Add the following method to the implementation:

| ----- |
|

    - (void)locationManager:(CLLocationManager *)manager didUpdateLocations:([NSArray][22] *)locations
    {
        // Last object contains the most recent location
        CLLocation *newLocation = [locations lastObject];
    &nbsp;
        // If the location is more than 5 minutes old, ignore it
        if([newLocation.timestamp timeIntervalSinceNow] &gt; 300)
            return;
    &nbsp;
        [self.locationManager stopUpdatingLocation];
    &nbsp;
        WeatherHTTPClient *client = [WeatherHTTPClient sharedWeatherHTTPClient];
        client.delegate = self;
        [client updateWeatherAtLocation:newLocation forNumberOfDays:5];
    }

 |

Now when there's an update to the user's whereabouts, you can call the singleton WeatherHTTPClient instance to request the weather for the current location.

Remember, WeatherHTTPClient has two delegate methods itself that you need to implement. Add the following two methods to the implementation:

| ----- |
|

    - (void)weatherHTTPClient:(WeatherHTTPClient *)client didUpdateWithWeather:(id)weather
    {
        self.weather = weather;
        self.title = @"API Updated";
        [self.tableView reloadData];
    }
    &nbsp;
    - (void)weatherHTTPClient:(WeatherHTTPClient *)client didFailWithError:([NSError][21] *)error
    {
        UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Error Retrieving Weather"
                                                            message:[[NSString][15] stringWithFormat:@"%@",error]
                                                           delegate:nil
                                                  cancelButtonTitle:@"OK" otherButtonTitles:nil];
        [alertView show];
    }

 |

When the WeatherHTTPClient succeeds, you update the weather data and reload the table view. In case of a network error, you display an error message.

Find the _apiTapped:_ method and replace it with the following:

| ----- |
|

    - (IBAction)apiTapped:(id)sender
    {
        [self.locationManager startUpdatingLocation];
    }

 |

Build and run your project (try your device if you have any troubles with your simulator), tap on the API button to initiate the WeatherHTTPClient request, and you should see something like this:

  
![World Weather Online API][40]  

Here's hoping your upcoming weather is as sunny as mine!

## I'm Not Dead Yet!

You might have noticed that this external web service can take some time before it returns with data. It's important to provide your users with feedback when doing network operations so they know the app hasn't stalled or crashed.

  
![][41]  

Luckily, AFNetworking comes with an easy way to provide this feedback: _AFNetworkActivityIndicatorManager_.

In _WTAppDelegate.m_, add this import just below the other:

| ----- |
|

    #import "AFNetworkActivityIndicatorManager.h"

 |

Then find the _application:didFinishLaunchingWithOptions:_ method and replace it with the following:

| ----- |
|

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:([NSDictionary][20] *)launchOptions
    {
        [AFNetworkActivityIndicatorManager sharedManager].enabled = YES;
        return YES;
    }

 |

Enabling the sharedManager automatically displays the network activity indicator whenever a new operation is underway. You won't need to manage it separately for every request you make.

Build and run, and you should see the little networking spinner in the status bar whenever there's a network request:

  
![Networking Spinner][42]  

Now there's a sign of life for your user even when your app is waiting on a slow web service.

## Downloading Images

If you tap on a table view cell, the app takes you to a detail view of the weather and an animation illustrating the corresponding weather conditions.

That's nice, but at the moment the animation has a very plain background. What better way to update the background than… over the network!

Here's the final AFNetworking trick for this tutorial: AFHTTPRequestOperation can also handle image requests by setting its _responseSerializer_ to an instance of _AFImageResponseSerializer_.

There are two method stubs in _WeatherAnimationViewController.m_ to implement. Find the _updateBackgroundImage:_ method and replace it with the following:

| ----- |
|

    - (IBAction)updateBackgroundImage:(id)sender
    {
        [NSURL][18] *url = [[NSURL][18] URLWithString:@"http://www.raywenderlich.com/wp-content/uploads/2014/01/sunny-background.png"];
        [NSURLRequest][19] *request = [[NSURLRequest][19] requestWithURL:url];
    &nbsp;
        AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
        operation.responseSerializer = [AFImageResponseSerializer serializer];
    &nbsp;
        [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
    &nbsp;
            self.backgroundImageView.image = responseObject;
            [self saveImage:responseObject withFilename:@"background.png"];
    &nbsp;
        } failure:^(AFHTTPRequestOperation *operation, [NSError][21] *error) {
    &nbsp;
            NSLog(@"Error: %@", error);
        }];
    &nbsp;
        [operation start];
    }

 |

This method initiates and handles downloading the new background. On completion, it returns the full image requested.

In WeatherAnimationViewController.m, you will see two helper methods, imageWithFilename: and saveImage:withFilename:, which will let you store and load any image you download. updateBackgroundImage: calls these helper methods to save the downloaded images to disk.

Find the _deleteBackgroundImage:_ method and replace it with the following:

| ----- |
|

    - (IBAction)deleteBackgroundImage:(id)sender
    {
    	[NSArray][22] *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    	[NSString][15] *path = [[paths objectAtIndex:0] stringByAppendingPathComponent:@"WeatherHTTPClientImages/"];
    &nbsp;
        [NSError][21] *error = nil;
        [[[NSFileManager][43] defaultManager] removeItemAtPath:path error:&amp;error];
    &nbsp;
        [NSString][15] *desc = [self.weatherDictionary weatherDescription];
        [self start:desc];
    }

 |

This method deletes the downloaded background image so that you can download it again when testing the application.

For the final time: build and run, download the weather data and tap on a cell to get to the detailed view. From here, tap the Update Background button. If you tap on a Sunny cell, you should see this:

  
![Sunny Day][44]  

## Where To Go From Here?

You can download the completed project from [here][45].

Think of all the ways you can now use AFNetworking to communicate with the outside world:

* AFHTTPOperation with AFJSONResponseSerializer, AFPropertyListResponseSerializer, or AFXMLParserResponseSerializer response serializers for parsing structured data
* UIImageView+AFNetworking for quickly filling in image views
* Custom AFHTTPSessionManager subclasses to access live web services
* AFNetworkActivityIndicatorManager to keep the user informed
* AFHTTPOperation with a AFImageResponseSerializer response serializer for loading images

The power of AFNetworking is yours to deploy!

If you have any questions about anything you've seen here, please pay a visit to the forums to get some assistance. I'd also love to read your comments!

[1]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/01/AFNetworking.png
[2]: /?page_id=9#scottsherwood
[3]: /?page_id=9#joshuagreene
[4]: /?p=51127
[5]: http://afnetworking.com/
[6]: /?p=21987
[7]: https://github.com/AFNetworking/AFNetworking
[8]: http://www.worldweatheronline.com/free-weather-feed.aspx "World Weather Online"
[9]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/WeatherStarter.zip
[10]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/Weather-App-Overview-700x381.png "Weather App Overview"
[11]: https://github.com/AFNetworking/AFNetworking "AFNetworking on GitHub"
[12]: http://cdn5.raywenderlich.com/wp-content/uploads/2013/12/afnetworking-folder-setup-512x500.png
[13]: http://cdn1.raywenderlich.com/wp-content/uploads/2013/12/include-afnetworking-700x396.png
[14]: http://cdn2.raywenderlich.com/wp-content/uploads/2013/02/AFNetworkingHero.jpg "AFNetworkingHero"
[15]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/
[16]: http://cdn3.raywenderlich.com/downloads/weather_sample.zip
[17]: https://en.wikipedia.org/wiki/JSON
[18]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/
[19]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSURLRequest_Class/
[20]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/
[21]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSError_Class/
[22]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/
[23]: http://www.jsonmodel.com/
[24]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSIndexPath_Class/
[25]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/afnetworking-json-retrieved-352x500.png "JSON Retrieved"
[26]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/afnetworking-plist-retrieved-352x500.png "PLIST Retrieved"
[27]: https://en.wikipedia.org/wiki/Simple_API_for_XML
[28]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSXMLParser_Class/
[29]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableDictionary_Class/
[30]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableString_Class/
[31]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSMutableArray_Class/
[32]: http://cdn3.raywenderlich.com/wp-content/uploads/2013/02/awww-yeah-female-aww-yeah-480x299.png "Aaaawwwww Yyyyeeeeaaa!"
[33]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/afnetworking-xml-retrieved-352x500.png "XML Retrieved"
[34]: http://www.raywenderlich.com/demos/weather_sample/weather.php?format=json "World Online Weather JSON Format Example"
[35]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPURLResponse_Class/
[36]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/UIImageView-AFNetworking-Example-352x500.png "UIImageView-AFNetworking Example"
[37]: http://rest.elkstein.org/2008/02/what-is-rest.html
[38]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/afhttpsessionmanager-get-post-700x378.png "AFHTTPSessionManager GET and POST"
[39]: http://developer.worldweatheronline.com/member/register
[40]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/01/World-Weather-Online-API-352x500.png "World Weather Online API"
[41]: http://www.raywenderlich.com/wp-content/uploads/2013/02/Needfeedback1-480x177.png "Needfeedback"
[42]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/networking-spinner.png
[43]: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSFileManager_Class/
[44]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/Sunny-Day-352x500.png "Sunny Day"
[45]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/Weather_Final.zip
  </nsxmlparserdelegate,></nsobject></weatherhttpclientdelegate></nsxmlparserdelegate,></nsxmlparserdelegate></dict></array></dict></dict>