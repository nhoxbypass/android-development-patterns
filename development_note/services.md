## Android Service

* Only [one instance](https://stackoverflow.com/questions/22909600/running-multiple-instance-of-a-service) of specific service is allow to run at the same time.
* The Service and IntentService may be triggered from any thread, activity or other application component.
* Use `bindService()` when need to communicate between activity and service (thru service connection).
* `START_STICKY` explicit started and stop aas needed, `START_NOT_STICKY` tell OS not to re-start service again when have enough memory (when service was killed before due to device run out of memory)

| Service        | IntentService          |
| -------------|---------------|
| used in tasks with no UI, but shouldn't be too long ( If need to perform long or heavy tasks, must use threads within Service) | used in long tasks usually with no communication to main UI thread |
| Main thread | New worker thread | 
| Block main thread | Cannot run task in parallel. Hence all the consecutive intents will go into the message queue for the worker thread and will execute sequentially |
| Must call `stopSelf()` or `stopService()` if not use bounded service | Do not have to call, auto shutdown after all start requests have been handled |

* For performance related stuffs of `IntentService` see [this](https://github.com/nhoxbypass/android-development-patterns-note/blob/master/performance_note.md#season-5-ep-07).

