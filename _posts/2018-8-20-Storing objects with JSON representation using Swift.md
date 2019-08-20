---
layout: post
title: Storing objects with JSON representation using Swift
---

Storing objects is always a major requirement for a mobile application. Although there are several long-er ways such as creating an sqlite database, usage of frameworks such as CoreData; just saving your objects as some kind of data representation directly will be enough for most of the apps. You can also use `NSKeyedArchiver / NSKeyedUnarchiver` for this kind of simple operation, I’m going to show a way to create a dictionary representation for an object and use that for storing. You might need that dictionaries for http requests or pretty logging later.  

Now first things first, let’s know our enemy classes that needs to be stored on disk. I have this tourist guide mobile application and I have created the following classes to keep the planned travel data I need:

    struct Place {
        let name: String
        let placeId: Int
    }
    class DailyTrip {
        var dailyPlaceList: [Place]!
        let date: Date!
    }
    class Trip {
      var dailyTrips: [DailyTrip]!
      let startDate: Date!
      let endDate: Date!
    }

Simple.  `Trip` class that has an array of `DailyTrip` instances which have `Place` lists. Consider `Trip` as the whole trip, and `DailyTrip` as just one day of that trip as their names also indicate. I didn’t add required initializers and functions that manipulates the trip objects for classes. Also you might wonder why I used class for trip instances and struct for place. Since it’s not the topic of this story, still there are great explanations online: <http://stackoverflow.com/questions/24232799/why-choose-struct-over-class>

Now we create a `Writer` class for the `Trip` class and make it responsible for read-write operations:

    class Writer {
        let trip: Trip // Writer is initialized with this trip
        func finalize() {
            do {
                let data = try JSONSerialization.data(withJSONObject:     self.trip.toJSON(), options: .prettyPrinted)
                let filePath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first
                try data.write(to: NSURL.fileURL(withPath: filePath!).appendingPathComponent("trip.json"))
            } catch {
                print("Error occured.")
            }
        }
    }

Simple, huh! But there are a couple of missing stuff here:

1. What if we want to store just a `Place` for some feature?
2. What if we change our decision, and want to store the JSON in UserDefaults / temporary folder etc. later?
3. Requirements might change such as we would need to store the file encrypted?

First one is too obvious, Writer shouldn’t have reference directly to `Trip`. Instead that variable should be a `JSONRepresentable` type.  

To fix second issue, we need to think a little deeper: with regard to single responsibility principle, what parameters Writer should know about to write a file?

1. JSON representable object 
2. Target URL. Let’s add a new variable to:  

        class Writer {
            let representable: JSONRepresentable
            let targetURL: URL
            // Update the initializer also to take 2 parameters
            func finalize() {
                do {
                    let data = try JSONSerialization.data(withJSONObject: self.representable.toJSON(), options: .prettyPrinted)
                    try data.write(to: targetURL)
                } catch {
                    print("Error occured.")
                }
            }
        }

Seems better, but how do we use this to write our file to Documents folder now? I will create a `DocumentsWriter` class for this, since there are limited target root folders for us it’s not a problem to have some small helper subclasses of Writer:

    class DocumentsWriter: Writer {
        init?(_ representable: JSONRepresentable?, filename: String) {
            let documentsPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first!
            let url = NSURL.fileURL(withPath: documentsPath).appendingPathComponent(filename)
            super.init(representable, targetURL: url)
        }
    }
    
Again if we ask ourselves about what `DocumentsWriter` should know? The answer is simple, just the path of Documents folder :) It will act as a proxy and forward that information to its super class which is `Writer`.  

What about this encryption thing? Who wants to encrypt some trip information which includes some touristic attractions and dates. To keep up the motivation; I’m assuming this app is written for some blonde magnificent president of an oil rich country.  

We can’t create a `EncryptedWriter` and use it as a subclass of `DocumentsWriter` this time. First we need a class that encrypts the `Data` and returns a new `Data` instance. But again, there are lots of encryption algorithms out there; we can’t directly allow whole class to implement an algorithm. Algorithms change. I created an `Encryptor` protocol and write my super encryption algorithm to a new encryptor class which conforms to it. Programming to interfaces (protocols) is critical to minimize coupling of our classes.

    protocol Encryptor {
        func encrypt(_ data: Data) -> Data
    }
    class UKEncryptor: Encryptor {
        func encrypt(_ data: Data) -> Data {
            var outputData = Data()
            for b in [UInt8](data) {
                outputData.append(b + 1)
            }
            return outputData
        }
    }
    
And update the `Writer` class as:

    class Writer {
        var encryptor: Encryptor?
        func finalize() {
            do {
                var data = try JSONSerialization.data(withJSONObject: self.representable.toJSON(), options: .prettyPrinted)
                if let encryptor = self.encryptor {
                    data = encryptor.encrypt(data)
                }
                try data.write(to: targetURL)
            } catch {
                print("Error occured.")
            }
        }
    }

Just assign your instance of `DocumentsWriter`’s encryptor variable to an `UKEncryptor` object. That’s all. Every class has it’s own responsibilities, and they are very open for extension.
Now there’s another requirement here, which is reading that file to create related objects. Same process, in reverse order.

*PS: Don’t use my encryption algorithm in your real projects :)*
