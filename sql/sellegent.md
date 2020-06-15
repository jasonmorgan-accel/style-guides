---
title: Notes on SQL in Selligent
category: SQL
order: 12
nav: true
---

## Overview

Selligent Marketing Cloud (SMC) is currently running on MS SQL Server 2016.  There are a number of things to consider when dealing with SQL in SMC.  We will explain these here.

In general all guide lines from (/sql) apply.

## Stored Prorcedures

Stored procedure names are required to begin with SP_.  While this going against general best practice according to (/sql), it is a requirement of the system.

## Cursors

The Selligent DBA team really do not like currsors.  So ,just do not write them.



