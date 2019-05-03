# auto-reconnect-rabbitmq
Simple **reconnect** logic for rabbitmq using amqplib 

## Resume
    If your broker dies for any reason this code will help either your producer or consumer to easily recconect without much problem.

    Basically, if that happen, every minute they`ll try to reconect until they succeed.

```javascript
const econnrefused = "ECONNREFUSED"
const handshakeError = "Socket closed abruptly during opening handshake"

async function connectRabbitMQ() {
  try {
    var conn = await amqp.connect(rabbitUrl)

    conn.on("error", function (err) {
      console.error("RabbitMQ onError error: " + err.message)
    })

    conn.on("close", function (err) {
      console.error("RabbitMQ onClose error: " + err.message)
      setTimeout(connectRabbitMQ, 60000)
    })

    var ch = await conn.createChannel()
    await ch.assertQueue(queue, { durable: true })
    await ch.prefetch(1)

    console.log(' [x] Awaiting messages [x] ')

    ch.consume(queue, (msg) => {
      console.log(" [x] Received %s", msg.content.toString())
    })

  } catch (err) {
    console.error("RabbitMQ error: " + err)
    if(err.code === econnrefused || err.message === handshakeError)
      setTimeout(connectRabbitMQ, 60000)
  }
}

connectRabbitMQ()
```

