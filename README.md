# rx-micro
Prototype functional reactive HTTP/WebSocket API framework.

```js
// service.js
import micro from 'micro'
import RxMicro from 'rx-micro'
import rx from 'rx'

// Root requests stream (from HTTP and WebSocket)
const request$ = micro(emitRequestToStream).http(3000)
/** Bind API definition to requests stream (this kicks off some
    magic where rx-micro would dynamically load modules from
    the filesystem based on the declared API (in template string).
    Each module loaded would export an async request handler function.
    RxMicro would automatically create new request streams for
    each endpoint. */
const api$ = request$.combine(Observable.once(`
  INDEX /projects
  VIEW /projects/:id
  CREATE /projects
  UPDATE /projects
  
  INDEX /builds
  VIEW /builds/:id
  CREATE /builds
  
  INDEX /deployments
  VIEW /deployments/:id
  CREATE /deployments
`))
/** Here we can do some niffty magic to handle basic micro service
    stuff like rate-limiting, logging, parallelization, and error-handling. */
api$
  .rateLimit(1000) // limit to 1000 requests/min - uses simple throttle() underneath
  .do(logToNewRelic)
  .distribute(8) // run this service with 8 threads - use forJoin() underneath
  .subscribe()
  .catch(handleException)
```

```js
// api/projects/view.js
import { send } from 'micro'
import database from '../services/database'

// This is the async handler for VIEW /projects/:id
export default async function(id, queryParams, { req, res }) {
  const data = await database.fetch('project', id)
  send(data)
}
```
