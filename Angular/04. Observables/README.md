# Observables
Observables are main and most common used data structure in angular. They were projected to handle data emits on time like soap, http and graphql requests, events and delegates and values emitting over time. They are also new standard for handling asynchronous operations for multiple values. To modify data we couldn't use normal js operations but special rxjs operators. Also while implemeting Observables we have to take care of unsubscribing them in order to avoid memory leak.
* Creating observables
  - of
  - from
  - http requests
* Kinds of observables
  - Observable
  - Subject
  - BehaviourSubject
  - ReplaySubject
  - AsyncSubject
* Operators
  - pipe
  - map
  - tap
  - filter
* Maps
  - switchMap
  - mergeMap,
  - concatMap,
  - exhaustMap
* Unsubscribing
  - async
  - take(1)
  - first
  - unsubscribe
  - next, complete
  - takeUntil
  - effects

### Resources
* [Observable vs Promise](https://www.syncfusion.com/blogs/post/angular-promises-versus-observables.aspx)
* [BehaviourSubject, ReplaySubject, AsyncSubject](https://luukgruijs.medium.com/understanding-rxjs-behaviorsubject-replaysubject-and-asyncsubject-8cc061f1cfc0)
* [async pipe vs subscribe](https://medium.com/angular-in-depth/angular-question-rxjs-subscribe-vs-async-pipe-in-component-templates-c956c8c0c794)
* [Unsubcribing](https://medium.com/angular-in-depth/the-best-way-to-unsubscribe-rxjs-observable-in-the-angular-applications-d8f9aa42f6a0)
* [switchMap, mergeMap, concatMap](https://luukgruijs.medium.com/understanding-rxjs-map-mergemap-switchmap-and-concatmap-833fc1fb09ff)
* [switchMap, mergeMap, flatMap, concatMap, exhaustMap](https://stackoverflow.com/questions/49698640/flatmap-mergemap-switchmap-and-concatmap-in-rxjs)
