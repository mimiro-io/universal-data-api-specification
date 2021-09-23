# Introduction

This specification defines a semantic graph data model, its serialisation and a simple and consistent API to expose, synchronise, update and query datasets. The specification defines how a server manages and exposes a collection of datasets. A dataset consists of entities. An entity has properties and references, and can be serialised as a JSON object. Clients can request all data in a dataset or just the entities that have changed since a given point in time. As well as consuming datasests, clients can also post updates to writeable datasets.

# Motivation

Many different APIs with different semantics and different data representations are constantly being defined and published. This requires developers to learn about and create different clients for the different endpoints when many of the core semantics remain the same. This specification looks to generalise the solution to publishing, consuming, synchronising and updating datasets.

# Background

The authors of this specification have been contributors to open standards for over twenty years. Some of the direct influences on this work are SDShare, RDF Net API, ISO Topic Maps Data Model, RDF, and OData.
