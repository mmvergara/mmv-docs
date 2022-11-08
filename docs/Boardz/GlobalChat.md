---
sidebar_position: 3
---

# Global Chat

Global chat is powered by **[Socket.io](https://socket.io/)** which makes setting up websockets easier

## Server Side

Setting up socket io is pretty straight forward, we first need to initialize it by triggering this function

```jsx
let io;
module.exports = {
  init: (httpServer) => {
    io = require("socket.io")(httpServer, {
      cors: {
        origin: "*",
        // for our globalchat we are only using this 2 methods
        methods: ["GET", "PUT", "DELETE"],
      },
    });
    return io;
  },
  getIO: () => {
    if (!io) throw new Error("Socket IO is not initialized");
    return io;
  },
};
```

Then establishing the connection to clients.

```jsx
mongoose.connect(process.env.MONGODB_URI).then(() => {
  const server = app.listen(process.env.PORT || 8080);
  const io = require("./utilities/socketIO").init(server);
  io.on("connection", (socket) => {
    console.log("Socket Connected to client" + socket.id || "(no id)");
  });
  console.log("Listening on Port" + process.env.PORT);
});
```

## Client Side

Using the `socket.io-client` package from **[Socket.io](https://socket.io/)** and useEffect Hook we can establish web socket conenction.

```jsx
import openSocket from "socket.io-client";
const socket = openSocket(API_URL);

useEffect(() => {
  socket.on("message", (messageData: { action: action, data: data }) => {
    if (messageData.action === "newMessage") {
      setMessageList((prev) => [...prev, messageData.data]);
      setTimeout(() => {
        liRef.current?.scrollIntoView({ behavior: "smooth" });
      }, 1);
    }
    if (messageData.action === "deletedMessage") {
      setMessageList((prev) => prev.filter((x) => x._id !== messageData.data._id));
    }
  });

  fetchMessageList();
  return () => {
    socket.off("message");
  };
}, []);
```

Sending and Deleting Message using axios

```jsx
export const putMessage = async (data: { message: string }): Promise<messageReqResponse> => {
  const result = (await AxiosRequest({
    method: "put",
    url: "/message/createnewmessage",
    data,
  })) as AxiosResponse<messageReqResponse>;
  return result.data;
};
export const deleteMessage = async ({ messageID }: { messageID: string }) => {
  const result = (await AxiosRequest({
    method: "delete",
    url: "/message/delete/" + messageID,
  })) as AxiosResponse<messageReqResponse>;
  return result.data;
};

```
