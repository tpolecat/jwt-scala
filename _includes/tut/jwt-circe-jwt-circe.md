## JwtCirce Object

### Basic usage

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |     expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond)
     |   , issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1636573962),None,Some(1478789202),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtCirce.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MzY1NzM5NjIsImlhdCI6MTQ3ODc4OTIwMn0.S1w3BxRzflVbFKedVcbsHWZuaLqh9DqJf8A0e16QDF0

scala> JwtCirce.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[io.circe.Json] =
Success({
  "exp" : 1636573962,
  "iat" : 1478789202
})

scala> JwtCirce.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1636573962),None,Some(1478789202),None))
```

### Encoding

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import cats.syntax.either._
import cats.syntax.either._

scala> import io.circe._, syntax._, jawn.{parse => jawnParse}
import io.circe._
import syntax._
import jawn.{parse=>jawnParse}

scala> import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val Right(claimJson) = jawnParse(s"""{"expires":${Instant.now.getEpochSecond}}""")
claimJson: io.circe.Json =
{
  "expires" : 1478789203
}

scala> val Right(header) = jawnParse( """{"typ":"JWT","alg":"HS256"}""")
header: io.circe.Json =
{
  "typ" : "JWT",
  "alg" : "HS256"
}

scala> // From just the claim to all possible attributes
     | JwtCirce.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNDc4Nzg5MjAzfQ.

scala> JwtCirce.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDc4Nzg5MjAzfQ.7SlnWRMf1AEem8BPYvCHQvfpY4GJH8dX_Kd8XQWI2v0

scala> JwtCirce.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDc4Nzg5MjAzfQ.7SlnWRMf1AEem8BPYvCHQvfpY4GJH8dX_Kd8XQWI2v0
```

### Decoding

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |     expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond)
     |   , issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1636573963),None,Some(1478789203),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtCirce.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MzY1NzM5NjMsImlhdCI6MTQ3ODc4OTIwM30.-g9-Z0iLUCjVQ8WwpnDQ1QC_A8_xJpSMmQf7McnSYdY

scala> // You can decode to JsObject
     | JwtCirce.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[io.circe.Json] =
Success({
  "exp" : 1636573963,
  "iat" : 1478789203
})

scala> JwtCirce.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(io.circe.Json, io.circe.Json, String)] =
Success(({
  "typ" : "JWT",
  "alg" : "HS256"
},{
  "exp" : 1636573963,
  "iat" : 1478789203
},-g9-Z0iLUCjVQ8WwpnDQ1QC_A8_xJpSMmQf7McnSYdY))

scala> // Or to case classes
     | JwtCirce.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1636573963),None,Some(1478789203),None))

scala> JwtCirce.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({},None,None,None,Some(1636573963),None,Some(1478789203),None),-g9-Z0iLUCjVQ8WwpnDQ1QC_A8_xJpSMmQf7McnSYdY))
```