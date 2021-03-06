dns4s
=====

dns4s is an implementation of the [DNS] protocol in [Scala].
It consists of the following parts:
* Core
* Akka IO Extension

Core
----
The core part contains the classes and objects used for en-/decoding of DNS
messages as well as an inner-[DSL] to con-/destruct DNS messages.

Akka IO Extension
-----------------
The akka part contains an akka-io extension.

Usage
-----
If you're using [sbt] just add the following to your build definition:
```scala
resolvers += "bintray" at "http://jcenter.bintray.com"

libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % "2.3.2",
  "com.github.mkroli" %% "dns4s-akka" % "0.3")
```

### Imports
Use the following imports to get started:
```scala
import akka.actor._
import akka.io.IO
import com.github.mkroli.dns4s.dsl._
import com.github.mkroli.dns4s.akka._
import scala.concurrent.duration._
```

### Server
The following is an excerpt from [examples/simple/../DnsServer.scala](https://github.com/mkroli/dns4s/blob/master/examples/simple/src/main/scala/com/github/mkroli/dns4s/examples/simple/DnsServer.scala):
```scala
class DnsHandlerActor extends Actor {
  override def receive = {
    case Query(_) ~ Questions(QName(host) ~ TypeA() :: Nil) =>
      sender ! Response ~ Questions(host) ~ Answers(ARecord("1.2.3.4"))
  }
}

object DnsServer extends App {
  implicit val system = ActorSystem("DnsServer")
  implicit val timeout = Timeout(5 seconds)
  IO(Dns) ? Dns.Bind(system.actorOf(Props[DnsHandlerActor]), 5354)
}
```

### Client
The following is an excerpt from [examples/simple-client/../DnsClient.scala](https://github.com/mkroli/dns4s/blob/master/examples/simple-client/src/main/scala/com/github/mkroli/dns4s/examples/simple/client/DnsClient.scala):
```scala
implicit val system = ActorSystem("DnsServer")
implicit val timeout = Timeout(5 seconds)
import system.dispatcher

IO(Dns) ? Dns.DnsPacket(Query ~ Questions("google.de"), new InetSocketAddress("8.8.8.8", 53)) onSuccess {
  case Response(Answers(answers)) =>
    answers.collect {
      case ARecord(arecord) => println(arecord.address.getHostAddress)
    }
}
```


[Scala]:http://www.scala-lang.org
[DNS]:http://en.wikipedia.org/wiki/Domain_Name_System
[DSL]:http://en.wikipedia.org/wiki/Domain-specific_language
[sbt]:http://scala-sbt.org/
