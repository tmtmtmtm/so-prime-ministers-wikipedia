Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item: https://www.wikidata.org/wiki/Q1335939
contains all the data expected already.

Step 2: Tracking page
=====================

PositionHolderHistory already exists; current version is
https://www.wikidata.org/w/index.php?title=Talk:Q1335939&oldid=1189699921
with 10 dated memberships and 11 undated; and 21 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = https://en.wikipedia.org/wiki/List_of_prime_ministers_of_Somalia

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Main tweak was to avoid the pre-independence office-holders, as that is
for a different office.

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

7 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/25e0de45f46aa

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

29 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/08cdd10a86811/

Step 8: Clean up bare P39s
==========================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' | xargs wd sparql bare-and-not-bare-P39.js |
      jq -r '.[] | "\(.bare_ps)"' | sort | uniq |
      wd rc --batch --summary "Remove bare P39s where qualified one exists"

-> https://tools.wmflabs.org/editgroups/b/wikibase-cli/14cc748a447a4/

Step 9: Refresh the Tracking Page
==================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q1335939&oldid=1239413814
