---
layout: post
title: "HOWTO Use Postgres Regexp Functions to Fix Data in a CLOB"
date: 2012-10-02 11:12
comments: true
categories: bbbb cloud bamboo postgres regex 
---
A bad copy and paste into a form field in Bamboo dropped a Unicode character where it shouldn't have, and blew up a project plan; the only real way to repair this was directly in the database:

Looking at the build table – `select build_id from build where full_key like 'SCALA-GRAPH-COMPILE';` – returns 1769482 as the build_id for the SCALA-GRAPH-COMPILE plan.

Next, confirm the bad data ... (nb. I took a shortcut here, in that I'd already scanned the data looking for the script I'd added):

    bamboo_production=# select build_definition_id, regexp_matches(xml_definition_data, '.&#0;zip') from build_definition;
    build_definition_id | regexp_matches 
    --------------------+----------------
                1835017 | {.&#0;zip}
    (1 row)

Finally, update the row using regexp_replace:

    bamboo_production=# update build_definition set xml_definition_data=(select regexp_replace(xml_definition_data, '.&#0;zip', '.zip') from build_definition where build_definition_id=1835017) where build_definition_id=1835017;
    UPDATE 1

PostgreSQL++.