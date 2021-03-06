Dart Redis Client
=================

A high-performance async/non-blocking Redis client for Dart. This project includes the 2 clients:

  - RedisClient - A high-level client with auto built-in serialization support for Dart's native types
  - RedisNativeClient - A low-level client providing raw bytes access to Redis's binary-safe string values

As all operations are async they return [Futures](http://api.dartlang.org/dart_core/Future.html) for better handling of asynchronous operations. 

### v1.0 Released
The Redis Client API is now feature complete with support for all ADMIN tasks as well as all KEYS, LISTS, SETS, SORTED SETS and HASH collections [including tests for all operations](https://github.com/mythz/DartRedisClient/blob/master/tests/RedisClientTests.dart).

Follow [@demisbellot](http://twitter.com/demisbellot) for project updates.

## Adding Dependencies with Pubspec 
    dependencies:
      dartredisclient:
        git: git://github.com/mythz/DartRedisClient.git

 
## Example Usage

      RedisClient client = new RedisClient("password@localhost:6379/0");

      var items = ["B","A","A","C","D","B","E"];
      var itemScores = {"B":2,"A":1,"C":3,"D":4,"E":5};

      client.smadd("setId", items);
      client.smembers("setId").then((members) => print("setId contains: $members"));
      client.mrpush("listId", items);
      client.lrange("listId").then((items) => print("listId contains: $items"));
      client.hmset("hashId", itemScores);
      client.hmget("hashId", ["A","B","C"]).then((values) => print("selected hashId values: $values"));
      client.zmadd("zsetId", itemScores);
      client.zrangeWithScores("zsetId", 1, 3).then((map) => print("ranked zsetId entries: $map"));
      client.zrangebyscoreWithScores("zsetId", 1, 3).then((map) => print("scored zsetId entries: $map"));

      var users = [{"name":"tom","age":29},{"name":"dick","age":30},{"name":"harry","age":31}];
      users.forEach((x) => client.set("user:${x['name']}", x['age']));
      client.keys("user:*").then((keys) => print("keys matching user:* $keys"));

      client.info.then((info) {
          print("Redis Server info: $info");
          print("Redis Client info: ${client.raw.stats}");
          start();
      });

Which generates the following output:

      setId contains: [A, B, C, D, E]
      listId contains: [B, A, A, C, D, B, E]
      selected hashId values: [1, 2, 3]
      ranked zsetId entries: {C: 3.0, D: 4.0, B: 2.0}
      scored zsetId entries: {A: 1.0, C: 3.0, B: 2.0}
      keys matching user:* [user:tom, user:dick, user:harry]

      Redis Server info: {# Server: null, redis_version: 2.5.9, ... db0: keys=7,expires=0}
      Redis Client info: {rewinds: 0, reads: 246, bytesRead: 3152, bufferWrites: 239, flushes: 14, bytesWritten: 740}

More examples can be found in 150+ tests in [RedisClientTests.dart](https://github.com/mythz/DartRedisClient/blob/master/tests/RedisClientTests.dart) - [latest testrun](https://gist.github.com/2698702).

## API

### RedisClient

A high-level interface with pluggable encoders/decoders providing API for native Dart types.

All methods with **Object** types allow you to pass and return any object, i.e. any int, double, bool, String, Date's persisted will return native int, double, bool, String, Date types out. Any complex types including Lists, Maps are serialized as JSON and auto-deserialized as well. Examples of this built-in serialization support are visible in the [JsonEncoderTests.dart](https://github.com/mythz/DartRedisClient/blob/master/tests/JsonEncoderTests.dart)

    interface RedisClient default _RedisClient {
        RedisClient([String connStr]);
        RedisNativeClient get raw();

        //ADMIN
        int get db();
        Future select(int db);
        Future<Date> get lastsave();
        Future<int> get dbsize();
        Future<Map> get info();
        Future flushdb();
        Future flushall();
        Future<bool> ping();
        Future<Object> echo(Object value);
        Future save();
        Future bgsave();
        Future shutdown();
        Future bgrewriteaof();
        Future quit();

        //KEYS
        Future<String> type(String key);
        Future<List<String>> keys(String pattern);
        Future<Object> get(String key);
        Future<List<Object>> mget(List<String> keys);
        Future<Object> getset(String key, Object value);
        Future set(String key, Object value);
        Future setex(String key, int expireInSecs, Object value);
        Future psetex(String key, int expireInMs, Object value);
        Future<bool> persist(String key);
        Future mset(Map map);
        Future<bool> msetnx(Map map);
        Future<bool> exists(String key);
        Future<int> del(String key);
        Future<int> mdel(List<String> keys);
        Future<int> incr(String key);
        Future<int> incrby(String key, int incrBy);
        Future<double> incrbyfloat(String key, double incrBy);
        Future<int> decr(String key);
        Future<int> decrby(String key, int decrBy);
        Future<int> strlen(String key);
        Future<int> append(String key, String value);
        Future<String> substr(String key, int fromIndex, int toIndex);
        Future<String> getrange(String key, int fromIndex, int toIndex);
        Future<String> setrange(String key, int offset, String value);
        Future<int> getbit(String key, int offset);
        Future<int> setbit(String key, int offset, int value);
        Future<String> randomkey();
        Future rename(String oldKey, String newKey);
        Future<bool> renamenx(String oldKey, String newKey);
        Future<bool> expire(String key, int expireInSecs);
        Future<bool> pexpire(String key, int expireInMs);
        Future<bool> expireat(String key, Date date);
        Future<bool> pexpireat(String key, Date date);
        Future<int> ttl(String key);
        Future<int> pttl(String key);

        //SET
        Future<List<Object>> smembers(String setId);
        Future<int> sadd(String setId, Object value);
        Future<int> smadd(String setId, List<Object> values);
        Future<int> srem(String setId, Object value);
        Future<Object> spop(String setId);
        Future<bool> smove(String fromSetId, String toSetId, Object value);
        Future<int> scard(String setId);
        Future<bool> sismember(String setId, Object value);
        Future<List<Object>> sinter(List<String> setIds);
        Future<int> sinterstore(String intoSetId, List<String> setIds);
        Future<List<Object>> sunion(List<String> setIds);
        Future<int> sunionstore(String intoSetId, List<String> setIds);
        Future<List<Object>> sdiff(String fromSetId, List<String> withSetIds);
        Future<int> sdiffstore(String intoSetId, String fromSetId, List<String> withSetIds);
        Future<Object> srandmember(String setId);

        //LIST
        Future<List<Object>> lrange(String listId, [int startingFrom, int endingAt]);
        Future<int> lpush(String listId, Object value);
        Future<int> mlpush(String listId, List<Object> values);
        Future<int> lpushx(String listId, Object value);
        Future<int> mlpushx(String listId, List<Object> values);
        Future<int> rpush(String listId, Object value);
        Future<int> mrpush(String listId, List<Object> values);
        Future<int> rpushx(String listId, Object value);
        Future<int> mrpushx(String listId, List<Object> values);
        Future ltrim(String listId, int keepStartingFrom, int keepEndingAt);
        Future<int> lrem(String listId, int removeNoOfMatches, Object value);
        Future<int> llen(String listId);
        Future<Object> lindex(String listId, int listIndex);
        Future lset(String listId, int listIndex, Object value);
        Future<Object> lpop(String listId);
        Future<Object> rpop(String listId);
        Future<Object> rpoplpush(String fromListId, String toListId);

        //SORTED SET
        Future<int> zadd(String setId, num score, Object value);
        Future<int> zmadd(String setId, Map<Object,num> scoresMap);
        Future<int> zrem(String setId, Object value);
        Future<int> zmrem(String setId, List<Object> values);
        Future<double> zincrby(String setId, num incrBy, Object value);
        Future<int> zrank(String setId, Object value);
        Future<int> zrevrank(String setId, Object value);
        Future<List<Object>> zrange(String setId, int min, int max);
        Future<Map<Object,double>> zrangeWithScores(String setId, int min, int max);
        Future<List<Object>> zrevrange(String setId, int min, int max);
        Future<Map<Object,double>> zrevrangeWithScores(String setId, int min, int max);
        Future<List<Object>> zrangebyscore(String setId, num min, num max, [int skip, int take]);
        Future<Map<Object,double>> zrangebyscoreWithScores(String setId, num min, num max, [int skip, int take]);
        Future<int> zremrangebyrank(String setId, int min, int max);
        Future<int> zremrangebyscore(String setId, num min, num max);
        Future<int> zcard(String setId);
        Future<double> zscore(String setId, Object value);
        Future<int> zunionstore(String intoSetId, List<String> setIds);
        Future<int> zinterstore(String intoSetId, List<String> setIds);

        //HASH
        Future<bool> hset(String hashId, String key, Object value);
        Future<bool> hsetnx(String hashId, String key, Object value);
        Future hmset(String hashId, Map<String,Object> map);
        Future<int> hincrby(String hashId, String key, int incrBy);
        Future<double> hincrbyfloat(String hashId, String key, double incrBy);
        Future<Object> hget(String hashId, String key);
        Future<List<Object>> hmget(String hashId, List<String> keys);
        Future<int> hdel(String hashId, String key);
        Future<bool> hexists(String hashId, String key);
        Future<int> hlen(String hashId);
        Future<List<String>> hkeys(String hashId);
        Future<List<Object>> hvals(String hashId);
        Future<Map<String,Object>> hgetall(String hashId);

        void close();
    }

### RedisNativeClient

A low-level client providing raw bytes access to Redis's binary-safe string values. 
You can also get access to this client from the high-level **RedisClient** above through the **raw** property, e.g:

    redis.raw.get(key) => List<int>

The RedisNativeClient API:

    interface RedisNativeClient default _RedisNativeClient {
        RedisNativeClient([String connStr]);

        Map get stats();

        int get db();
        Future select(int db);
        Future<int> get dbsize();
        Future<int> get lastsave();
        Future flushdb();
        Future flushall();
        Future<Map> get info();
        Future<bool> ping();
        Future<List<int>> echo(List<int> value);
        Future save();
        Future bgsave();
        Future shutdown();
        Future bgrewriteaof();
        Future quit();

        //KEYS
        Future<String> type(String key);
        Future<List<String>> keys(String pattern);
        Future<List<int>> get(String key);
        Future<List<List<int>>> mget(List<String> keys);
        Future<List<int>> getset(String key, List<int> value);
        Future set(String key, List<int> value);
        Future setex(String key, int expireInSecs, List<int> value);
        Future psetex(String key, int expireInMs, List<int> value);
        Future<bool> persist(String key);
        Future mset(List<List<int>> keys, List<List<int>> values);
        Future<bool> msetnx(List<List<int>> keys, List<List<int>> values);
        Future<bool> exists(String key);
        Future<int> del(String key);
        Future<int> mdel(List<String> keys);
        Future<int> incr(String key);
        Future<int> incrby(String key, int count);
        Future<double> incrbyfloat(String key, double count);
        Future<int> decr(String key);
        Future<int> decrby(String key, int count);
        Future<int> strlen(String key);
        Future<int> append(String key, List<int> value);
        Future<List<int>> substr(String key, int fromIndex, int toIndex);
        Future<List<int>> getrange(String key, int fromIndex, int toIndex);
        Future<List<int>> setrange(String key, int offset, List<int> value);
        Future<int> getbit(String key, int offset);
        Future<int> setbit(String key, int offset, int value);
        Future<List<int>> randomkey();
        Future rename(String oldKey, String newKey);
        Future<bool> renamenx(String oldKey, String newKey);
        Future<bool> expire(String key, int expireInSecs);
        Future<bool> pexpire(String key, int expireInMs);
        Future<bool> expireat(String key, int unixTimeSecs);
        Future<bool> pexpireat(String key, int unixTimeMs);
        Future<int> ttl(String key);
        Future<int> pttl(String key);

        //SET
        Future<List<List<int>>> smembers(String setId);
        Future<int> sadd(String setId, List<int> value);
        Future<int> smadd(String setId, List<List<int>> values);
        Future<int> srem(String setId, List<int> value);
        Future<List<int>> spop(String setId);
        Future<bool> smove(String fromSetId, String toSetId, List<int> value);
        Future<int> scard(String setId);
        Future<bool> sismember(String setId, List<int> value);
        Future<List<List<int>>> sinter(List<String> setIds);
        Future<int> sinterstore(String intoSetId, List<String> setIds);
        Future<List<List<int>>> sunion(List<String> setIds);
        Future<int> sunionstore(String intoSetId, List<String> setIds);
        Future<List<List<int>>> sdiff(String fromSetId, List<String> withSetIds);
        Future<int> sdiffstore(String intoSetId, String fromSetId, List<String> withSetIds);
        Future<List<int>> srandmember(String setId);

        //LIST
        Future<List<List<int>>> lrange(String listId, int startingFrom, int endingAt);
        Future<int> lpush(String listId, List<int> value);
        Future<int> mlpush(String listId, List<List<int>> values);
        Future<int> lpushx(String listId, List<int> value);
        Future<int> mlpushx(String listId, List<List<int>> values);
        Future<int> rpush(String listId, List<int> value);
        Future<int> mrpush(String listId, List<List<int>> values);
        Future<int> rpushx(String listId, List<int> value);
        Future<int> mrpushx(String listId, List<List<int>> values);
        Future ltrim(String listId, int keepStartingFrom, int keepEndingAt);
        Future<int> lrem(String listId, int removeNoOfMatches, List<int> value);
        Future<int> llen(String listId);
        Future<List<int>> lindex(String listId, int listIndex);
        Future lset(String listId, int listIndex, List<int> value);
        Future<List<int>> lpop(String listId);
        Future<List<int>> rpop(String listId);
        Future<List<int>> rpoplpush(String fromListId, String toListId);

        //SORTED SETS
        Future<int> zadd(String setId, num score, List<int> value);
        Future<int> zmadd(String setId, List<List<int>> scoresAndValues);
        Future<int> zrem(String setId, List<int> value);
        Future<int> zmrem(String setId, List<List<int>> values);
        Future<double> zincrby(String setId, num incrBy, List<int> value);
        Future<int> zrank(String setId, List<int> value);
        Future<int> zrevrank(String setId, List<int> value);
        Future<List<List<int>>> zrange(String setId, int min, int max);
        Future<List<List<int>>> zrangeWithScores(String setId, int min, int max);
        Future<List<List<int>>> zrevrange(String setId, int min, int max);
        Future<List<List<int>>> zrevrangeWithScores(String setId, int min, int max);
        Future<List<List<int>>> zrangebyscore(String setId, num min, num max, [int skip, int take]);
        Future<List<List<int>>> zrangebyscoreWithScores(String setId, num min, num max, [int skip, int take]);
        Future<List<List<int>>> zrevrangebyscore(String setId, num min, num max, [int skip, int take]);
        Future<List<List<int>>> zrevrangebyscoreWithScores(String setId, num min, num max, [int skip, int take]);
        Future<int> zremrangebyrank(String setId, int min, int max);
        Future<int> zremrangebyscore(String setId, num min, num max);
        Future<int> zcard(String setId);
        Future<double> zscore(String setId, List<int> value);
        Future<int> zunionstore(String intoSetId, List<String> setIds);
        Future<int> zinterstore(String intoSetId, List<String> setIds);

        //HASH
        Future<bool> hset(String hashId, String key, List<int> value);
        Future<bool> hsetnx(String hashId, String key, List<int> value);
        Future hmset(String hashId, List<List<int>> keys, List<List<int>> values);
        Future<int> hincrby(String hashId, String key, int incrBy);
        Future<double> hincrbyfloat(String hashId, String key, double incrBy);
        Future<List<int>> hget(String hashId, String key);
        Future<List<List<int>>> hmget(String hashId, List<String> keys);
        Future<int> hdel(String hashId, String key);
        Future<bool> hexists(String hashId, String key);
        Future<int> hlen(String hashId);
        Future<List<String>> hkeys(String hashId);
        Future<List<List<int>>> hvals(String hashId);
        Future<List<List<int>>> hgetall(String hashId);

        void close();
    }

## Redis Connection Strings
The redis clients above take a single connection string containing the password, host, port and db in the following formats:

    pass@host:port/db
    pass@host:port
    pass@host
    host
    null => localhost:6379/0

Valid example:
    
    RedisClient client = new RedisClient("password@localhost:6379/0");

## RoadMap

### v2.0 release (2-4 weeks)
After the v1.0 release we'll start work on implementing the remaining functionality:
  - Transactions, Pub/Sub, Lua/Scripts

### v3.0 release (future)
Adding automatic failover support in v3.0, sharding, fast RPC direct-pipeline using the Redis wire format with node.js/C# processes

## Contributors

  - [mythz](https://github.com/mythz) (Demis Bellot)
  - [financeCoding](https://github.com/financeCoding) (Adam Singer)


### Feedback 

Feedback and contributions are welcome.

