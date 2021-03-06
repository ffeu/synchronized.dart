# synchronized

[![Build Status](https://travis-ci.org/tekartik/synchronized.dart.svg?branch=master)](https://travis-ci.org/tekartik/synchronized.dart)

Basic lock mechanism to prevent concurrent access to asynchronous code

## Goal

You were missing hard to debug deadlocks, here it is! 

The goal is to propose a solution similar to critical sections and offer a simple `synchronized` API à la Java style.
It provides a basic Lock/Mutex solution to allow features like transactions.

The name is biased as we are single threaded in Dart. However since we write asychronous code (await) like we would
write synchronous code, it makes the overall API feel the same.

The goal is to ensure for a single process (single isolate) that some asynchronous operations can run
without conflict. It won't solve cross-process (or cross-isolate) synchronization.

For single process (single isolate) accessing some resources (database..), it can help to
 * Provide transaction on database system that don't have transaction mechanism (mongodb, file system)
 * In html application make sure some asynchronous UI operation are not conflicting (login, transition)

## Feature

 * Synchronized block are reentrant
 * Timeout support
 * Support for non-reentrant lock (not using Zone)
 * Consistent behavior (i.e. if it is unlocked calling synchronized grab the lock)
 * Values and Errors are properly reported to the caller
 * Work on Browser, DartVM and Flutter
 * No dependencies (other than the sdk itself)
 
It differs from the `pool` package used with a resource count of 1 by supporting a reentrant option

## Usage

A simple usage example:

    import 'package:synchronized/synchronized.dart';

    main() async {
      var lock = new Lock();
      await lock.synchronized(() async {
        // Only this block can run (once) until done 
        ...
      });
    }
    
Any object can become a locker, so in a class method you can use

    synchronized(this, () async {
      // do some stuff
    });

If you need a re-entrant lock you can use

````
var lock = new Lock(reentrant: true);
await lock.synchronized(() async {
  // do some stuff
  // ... 
  
  await lock.synchronized(() async {
    // other stuff
  }
});
````
        
A basic lock is not reentrant by default and does not use Zone. It behaves like an async executor with a pool capacity
of 1

````
var lock = Lock();
lock.synchronized(() async {
  // do some stuff
  // ...
});
````
    
The return value is preserved

    int value = await lock.synchronized(() {
      return 1;
    });
    
## How it works

Each `Lock` object keep a list of pending tasks. The next tasks is executed once the previous one is done

Re-entrant locks uses `Zone` to know in which context a block is running in order to be reentrant.

## Example

Consider the following dummy code

    Future writeSlow(int value) async {
      new Future.delayed(new Duration(milliseconds: 1));
      stdout.write(value);
    }
    
    Future write(List<int> values) async {
      for (int value in values) {
        await writeSlow(value);
      }
    }
    
    Future write1234() async {
      await write([1, 2, 3, 4]);
    }

Doing 

    write1234();
    write1234();
    
would print

    11223344
    
while doing

    lock.synchronized(write1234);
    lock.synchronized(write1234);

would print

    12341234

## Features and bugs

Please feel free to: 
* file feature requests and bugs at the [issue tracker][tracker]
* or [contact me][contact_me]
* [How to][how_to] guide


[tracker]: https://github.com/tekartik/synchronized.dart/issues
[contact_me]: http://contact.tekartik.com/
[how_to]: https://github.com/tekartik/synchronized.dart/blob/master/doc/how_to.md

