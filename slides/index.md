- title : Websocket Injection into a Fable-Elmish WebApp
- description : Using websockets in a Fable-Elmish app to allow for external event injection
- author : Justin Sacks
- theme : blood
- transition : default

***

### WebSocket Injection into a Fable-Elmish WebApp

<img style="border: none; height:2em; box-shadow:none;" src="images/Prolucid.png" alt="Prolucid logo" />

<br/>

Justin Sacks

***

### WebSocket Injection into a Fable-Elmish WebApp

- Traditional Events
- Subscription Events
- WebSockets
- Examples

***

### Traditional Events

- Dispatch originates from view 
  - e.g. button click
- Subsequent side-effect messages can cascade

---

### Traditional Events

#### Dispatch from view

    let view model dispatch =
        R.div [] [
            R.button [ OnClick (fun _ -> dispatch Msg1) ] [ unbox "Click Me" ]
        ]

#### Side-effect

    let update (msg:Msg) model =
        match msg with
        | Msg1 -> model, Cmd.ofMsg Msg2
        | Msg2 -> ...

***

### Subscription Events

- Result of external event
  - e.g. timer
- `Program.withSubscription` allows subscriptions to call dispatch when they need to

---

### Subscription Events
    let timerTick dispatch =
        let timer = new System.Timers.Timer 1000.
        timer.Elapsed.Subscribe (fun _ -> dispatch Msg1) |> ignore
        timer.Enabled <- true

    let subscribe model = Cmd.ofSub timerTick

    Program.mkProgram init update view
    |> Program.withSubscription subscribe
    |> Program.withReact "elmish-app"
    |> Program.run

***

### WebSockets

- Standardized by IETF as RFC 6455
- Describes persistent connection between client (including but not limited to browser) and server
- Bidirectional
- Event-driven

---

### WebSockets (Client)

#### Fable Bindings

    type [<AllowNullLiteral>] WebSocket =
        ...
        abstract onmessage: Func<MessageEvent, obj> with get, set

    type [<AllowNullLiteral>] WebSocketType =
        ...
        [<Emit("new $0($1...)")>] abstract Create: 
            url: string * ?protocols: U2<string, ResizeArray<string>> 
            -> WebSocket

---

### WebSockets (Client)
    // Shared messages between server and client
    type WsMessage =
        | SomethingHappened of string

    let ws = WebSocket.Create("ws://" + window.location.hostname + ":8080")

    let onMessage dispatch =
        fun (msg: MessageEvent) ->
            let msg' = msg.data |> string |> ofJson<WsMessage>
            // Dispatch a local client message
            match msg' with
            | SomethingHappened e -> dispatch <| Msg1(e)

    let wsCallbacks dispatch =
        ws.onmessage <- unbox (onMessage dispatch)
    
    let subscribe model = Cmd.ofSub wsCallbacks

---

### WebSockets (Server)

- Can't use browser implementation
- Many server-specific implementations available
- fable-elmish sample (counter-ws)
  - Uses `fable-import-ws`
  - NodeJS (Express) server written in F#!

***

### Examples

- Counter-ws
- Circle War

---

### Examples - Counter-ws

- Modified version of fable-elmish counter sample
- Server counts for you

---

### Examples - Circle War

- Click to create your own circles and destroy other players' circles
- Server keeps track of players and circles
- Broadcasts state changes to all clients

---

### Examples - Circle War

#### Shared messages
    
    type PlayerId = int
    type WsMessage =
        | IdPlayer of pid:PlayerId
        | PlayerJoined of pid:PlayerId
        | PlayerLeft of pid:PlayerId
        | DeleteCircle of pid:PlayerId * x:float * y:float
        | AddCircle of pid:PlayerId * x:float * y:float

---

### Examples - Circle War

<img style="border: none; box-shadow:none" src="images/CircleWar.png" alt="Sequence diagram" />

---

### Examples - Circle War

#### Client message handling

    // Local client messages
    type Msg =
        ...
        | Send of WsMessage
        | Rcv of WsMessage

    let onMessage dispatch =
        fun (msg: MessageEvent) ->
            msg.data |> string |> ofJson |> Rcv |> dispatch

    let send msg = toJson msg |> ws.send

***

### Questions

?