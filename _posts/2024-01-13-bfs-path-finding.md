---
layout: post
title:  "Graphs: finding paths in unweighted graphs"
date:   2024-01-05 22:22:22 +0100
categories: computer-science
excerpt_separator: <!--more-->
excerpt: graphs
tags: graph scala fp shortest path directed undirected unweighted
author: Paweł Zabczyński
---

Searching is one of the most common operations that can be done in many contexts. In this article, I will briefly go through a breadth-first search use case: shortest path finding in an unweighted graph.

First of all, let's define what the shortest path is in the context of an unweighted graph.

> Path is a sequence of vertices where each vertex is distinct


> Shortest path is a path that contains the minimal possible number of vertices


Let's start with an example graph:

![image](/assets/images/graphs/unweigth_graph_shortest_path.PNG)

Let's find paths from node `A` to `G`. Possible paths are:

- `A -> B -> E -> G`
- `A -> C -> F -> G`
- `A -> D -> G`

For the given example there are three paths, but one is shorter than the others. Now let's define a simple procedure for how to find it.

1. go to start node, mark it as visited, add to path and push into queue
2. pop path from queue and peek last added node u
3. take all u descendant nodes
4. if descendant node is visited, skip
5. if descendant node is a target add to path and return new path
6. if not equals target node, mark as visited, construct new path with path and node
7. new path push into queue
8. go to point 2 and continue until find path or queue will be empty


Now let's write code in Scala that implements this algorithm. At the beginning, let's implement it in an imperative way using mutable collections:

```scala
import scala.collection.mutable.{Queue => MQueue, Set => MSet}

def shortestPathIterative(
      start: String,
      end: String,
      graph: Map[String, List[String]]
  ): List[String] = {
    val que     = MQueue(List(start)) // 1
    val visited = MSet.empty[String]

    while (que.nonEmpty) {
      val path = que.dequeue() // 2
      val u    = path.head // 2
      visited.add(u) // 1
      for (v <- graph(u)) { // 3
        if (v == end) {
          return (v :: path).reverse // 5
        } else if (!visited(v)) {
          visited.add(v) // 6
          que.addOne(v :: path) // 6, 7
        }
      } // 4
    }

    Nil
  }
```


Now that the iterative algorithm, which is much more straightforward, is done, let's translate it into a more FP style with immutable structures and tail recursion:

```scala
import scala.annotation.tailrec

def shortestPathTailRec(
      start: String,
      end: String,
      graph: Map[String, List[String]]
  ): List[String] = {
    @tailrec
    def loop(
        queue: List[List[String]],
        visited: Set[String]
    ): List[String] = {
      queue match {
        case Nil => Nil
        case path :: tail => // 2
          path match {
            case u :: _ if u == end => path.reverse // 5
            case u :: _ =>
              graph(u).filterNot(visited) match { // 3, 4
                case Nil => loop(tail, visited)
                case descendants =>
                  loop(
                    tail ++ descendants.map(_ :: path), // 6, 7
                    visited ++ descendants // 6
                  )
              }
          }
      }
    }
    loop(List(List(start)), Set.empty) // 1
  }
```


Summarizing both implementations, the FP version looks more sophisticated and requires much more intellectual effort to achieve the same result. However, in my opinion it is much more elegant :)



[For full code check GitHub](https://github.com/sezerp/blog-code-scala/blob/master/src/main/scala/com/pawelzabczynski/graph/ShortestPathUnweightedGraph.scala)