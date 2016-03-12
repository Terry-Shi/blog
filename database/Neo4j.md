Neo4j

### Why Neo4j/graph database ? ###
#### How to store social network data in RDBMS?

----------

Social network such as twitter has a set of users who can follow each other.

User1 ---follow---> User2
 
User3 ---follow---> User5

User4 ---follow---> User1

----------
Simplified table structure

t_user Table
<table>
    <tr>
        <td>USER_ID</td>
        <td>NAME</td>
    </tr>
    <tr>
        <td>1</td>
        <td>John</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Kate</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Aleksa</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Jack</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Jonas</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Anne</td>
    </tr>
</table>

t_user_friend Table
<table>
    <tr>
        <td>ID</td>
        <td>USER_1</td>
        <td>USER_2</td>
    </tr>
    <tr>
        <td>1000</td>
        <td>1</td>
        <td>2</td>
    </tr>
    <tr>
        <td>1001</td>
        <td>3</td>
        <td>5</td>
    </tr>
    <tr>
        <td>1002</td>
        <td>4</td>
        <td>1</td>
    </tr>
    <tr>
        <td>1003</td>
        <td>6</td>
        <td>2</td>
    </tr>
    <tr>
        <td>1004</td>
        <td>4</td>
        <td>5</td>
    </tr>
    <tr>
        <td>1005</td>
        <td>1</td>
        <td>4</td>
    </tr>
</table>

#### Querying graph data using SQL：

Getting the direct friends of a 
particular user.
<pre>
select distinct uf.user_2 
  from t_user_friend uf 
 where uf.user_1 = ?
</pre>
- - - - - - - -
finding all friends of a user’s friends

<pre>
select distinct uf2.user_2 
  from t_user_friend uf1 inner joint t_user_friend uf2 on uf1.user_2 = uf2.user_1 
 where uf1.user_1 = ?
</pre>
----
suggest people from your friendship network as potential friends or contacts up to a certain degree of separation, or depth. If you wanted to do something similar to find friends of friends of friends of a user, you’d need another join operation:
<pre>
select distinct uf3.user_2 
  from t_user_friend uf1 inner joint t_user_friend uf2 on uf1.user_2 = uf2.user_1 
                         inner joint t_user_friend uf3 on uf2.user_2 = uf3.user_1 
 where uf1.user_1 = ?
</pre>
------
Querying graph data in Neo4j
<pre>
TraversalDescription traversalDescription = Traversal.description()
.relationships(“FOLLOW”, Direction.OUTGOING)
.evaluator(Evaluators.atDepth(2))
.uniqueness(Uniqueness.NODE_GLOBAL);
Iterable<Node> nodes = traversalDescription.traverse(nodeById).nodes();
</pre>

------

### Finding shortest path between start node and end node.
1. Pathing and Maps:

navigation units (ala Garmin, TomTom), that supply road directions from your current location to another, utilize graphs and advanced pathing algorithms.

<pre>
 PathFinder<Path> finder = GraphAlgoFactory.shortestPath(
     Traversal.expanderForTypes(ExampleTypes.MY_TYPE, Direction.OUTGOING), 15);
 Iterable<Path> paths = finder.findAllPaths(startNode, endNode);
</pre>

Ref:
http://docs.neo4j.org.cn/tutorials-java-embedded-graph-algo.html


2. Constraint Satisfaction:

A common problem in AI is to find some goal that satisfies a list of constraints. For example, for a University to set it's course schedules, it needs to make sure that certain courses don't conflict, that a professor isn't teaching two courses at the same time, that the lectures occur during certain time slots, and so on. Constraint satisfaction problems like this are often modeled and solved using graphs.

Ref：
http://stackoverflow.com/questions/703999/what-are-good-examples-of-problems-that-graphs-can-solve-better-than-the-alterna?lq=1

http://docs.neo4j.org.cn/tutorials.html

* Traversal is the query in graph database!
* Neo4j: The ACID-compliant database
* neo4j提供了两种遍历的方式：一种是深度搜索，第二种是广度搜索。也提供了两种搜索算法，一种是A*算法，第二种是dijkstra算法。提高了编程人员的工作效率。同时neo4j也有索引的功能，方便了多节点的查找。

Ref:
[dijkstra算法](http://zh.wikipedia.org/wiki/%E8%BF%AA%E7%A7%91%E6%96%AF%E5%BD%BB%E7%AE%97%E6%B3%95)

http://docs.jboss.org/drools/release/6.0.0.Beta5/optaplanner-docs/html_single/index.html
