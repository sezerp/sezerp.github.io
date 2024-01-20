---
layout: single
classes: wide
title:  "Graphs: finding paths in unweighted graphs"
date:   2024-01-05 22:22:22 +0100
categories: computer-science
excerpt_separator: <!--more-->
excerpt: graphs
tags: graph scala fp shorthest path directed undirected unweighted
author: Paweł Zabczyński
---

Searching is on of the most common operation that can be done in lot of contexts. In article, I wll brithly go throw breadth first search use case, in shorthest path finding in unweigth graph.

First of all, lets define what is shorthest path. In background of unweighted graph. 

> Path is a sequence of vertices where each vertex is distinct


> Shorthest path is a path that contains minimal posible number vertices


Lets start with example graph:

![image](/assets/images//graphs/unweigth_graph_shorthest_path.PNG)

Lets find paths from node `A` to `G`. Possible paths are:

- `A -> B -> E -> G`
- `A -> C -> F -> G`
- `A -> D -> G`

For given example exists three paths, but one is shorthest then other. Now lets define simple procedure how to find it.

1. go to start node, mark it as visited, add to path and push into queue
2. pop path from queue and peek last added node u
3. take all u descendant nodes
4. if descendant node is visited, skip
5. if descendant node is a target add to path and return new path
6. if not equals target node, mark as visited, construct new path with path and node
7. new path push into queue
8. go to point 2 and continue until find path or queue will be empty


Now lets write code in scala that implements this algorith. At the beggining lets implement it in imperative way using mutable collections:

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


Now when iterative algorith which is much strait forward, lets translate it into more FP with immutable strucutures and tail recursion:

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


Summarising both implementation the FP version looks more sophisticated and require much more intelectual effort to achive the same result. However, in my opinon much more elegant :) 



[For full code check GitHub](https://github.com/sezerp/blog-code-scala/blob/master/src/main/scala/com/pawelzabczynski/graph/ShortestPathUnweightedGraph.scala)