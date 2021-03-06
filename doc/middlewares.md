Middlewares
===========

Middlewares allow to easily hookup to different places in a messages handling
lifecycle. This document describes the available events.

## Events

### `pre_handle`

```
pre_handle(request: Request, responder: Responder)
```

Called before all the handling logic happened (before triggering the triggers
and so on), but inside the message handling lock.

### `flush`

```
flush(request: 'Request', stacks: List[Stack])
```

That's when messages are being flushed to the platform. This is an opportunity
to alter the stack to your liking.

### `resolve_trans_params`

```
resolve_trans_params(params: Dict[Text, Any], request: Request)
```

Do whatever processing you'd like to on the params. The default behaviour
of BERNARD is to render any StringToTranslate into actual strings so you can
use a translation as param. Example use:

```python
class AddName(BaseMiddleware):
    async def resolve_trans_params(self, params, request):
        params = await self.next(params, request)
        params['name'] = await request.user.get_friendly_name()
        return params
```

### `make_trans_flags`

Allows projects to add their own translation flags, like the user's gender.

```python
class AddGender(BaseMiddleware):
    async def make_trans_flags(self, request):
        flags = await self.next(request)
        flags['gender'] = (await request.user.get_gender()).value
        return flags
```

### `api_postback_me`

```
api_postback_me(msg: BaseMessage, platform: Platform)
```

When `bernard.js` gets information about the current user, by default
it will return something like

```javascript
{
    friendly_name: 'Foo',
    locale: 'en_US',
    platform: 'facebook',
}
```

You might want more information, like the user's gender. In this case
you can easily do the following:

```python
class AddGender(BaseMiddleware):
    async def api_postback_me(self, msg: BaseMessage, platform: Platform):
        info = await self.next(msg, platform)
        info['gender'] = (await msg.get_user().get_gender()).value
        return info
```

If you need an authentication token to your API, it's also the place
to communicate it to your webview.
