### A walk through of Solr

#### Single term search
```
A single term is a single word such as "test" or "hello"
q = PROVIDER_NAME.string: Jordan
q = PROVIDER_NAME.string: Nestor Jordan
```

#### Multiple term search
```
# A phrase is a group of words surrounded by double quotes such as "hello dolly"
q = PROVIDER_NAME.string: "Nestor, Jordan"
q = TITLE.string:  "follow up" # match both "follow up" and "follow-up" b/c follow-up will be tokenized as "follow" and "up"
```

#### Wildcard searches - only works for single term
```
# The search string te?t would match both test and text.
# The wildcard search: tes* would match test, testing, and tester. 
q = PROVIDER_NAME.string: Ka*i
q = PROVIDER_NAME.string: Ka?i
```

#### Fuzzy searches - based on Levenshtein distance
```
# To perform a fuzzy search, use the tilde ~ symbol at the end of a single-word term. 
q = PROVIDER_NAME.string: Josdan~1
# also works with phase
q = PROVIDER_NAME.string: "Nestor Jordan"~3 
```

#### Proximity searches - looks for terms that are within a specific distance
```
# to search for a "apache" and "jakarta" within 10 words of each other in a document, use the search: "jakarta apache"~10
q = PROVIDER_NAME.string: "Jordan Nestor"~2
```

#### Ranges searches - specifies a range of values
```
# Square brackets [ & ] denote an inclusive range query that matches values including the upper and lower bound.
# Curly brackets { & } denote an exclusive range query that matches values between the upper and lower bounds, but excluding the upper and lower bounds themselves.
# You can mix these types so one end of the range is inclusive and the other is exclusive. Here’s an example: count:{1 TO 10]
q = TIME_STR_KEY.long:[1450193520000 TO 1450193530000] 
```

#### Boosting a term - put an emphasize on a term
```
q = PROVIDER_NAME.string: Nestor^10 Jordan
q = TITLE.string: "Nephrology Consult"^4 "follow-up"
```

#### Constant score - when you only care about matches for a particular clause and don’t want other relevancy factors
```
q = (TITLE.string: follow up)^=1.0 # you can find the rank is different for q = TITLE.string:  follow up
```

#### Field combination - specifying multiple fields
```
q = PROVIDER_NAME.string: Jordan AND TITLE.string: follow up
# Important The field is only valid for the term that it directly precedes. so the query title:Do it right will find only "Do" in the title field. It will find "it" and "right" in the default field (in this case the text field).
```

#### Boolean Operators - by default it is OR
```
# It is important to specifiy the field between Boolen operators
# When specifying Boolean operators with keywords such as AND or NOT, the keywords must appear in all uppercase.
q = PROVIDER_NAME.string: WHITE # returns 77495
q = PROVIDER_NAME.string: JORDAN # 86193
q = PROVIDER_NAME.string: JORDAN OR PROVIDER_NAME.string: WHITE # 160466
q = +PROVIDER_NAME.string: JORDAN OR PROVIDER_NAME.string: WHITE # 86193. JORDAN is required
q = PROVIDER_NAME.string: JORDAN OR PROVIDER_NAME.string: WHITE# 3222 = 77495 + 86193 - 160466
q = PROVIDER_NAME.string: JORDAN NOT PROVIDER_NAME.string: WHITE # 82971 = 86193 - 3222
q = PROVIDER_NAME.string: JORDAN  -PROVIDER_NAME.string: WHITE # 82971, no WHITE is required
```

#### Escaping special characters - rare used. check https://lucene.apache.org/solr/guide/6_6/the-standard-query-parser.html#TheStandardQueryParser-EscapingSpecialCharacters

#### group terms into sub queries
```
# group the Boolean clauses within parentheses.
q = (PROVIDER_NAME.string: Jordan) AND (TITLE.string: (follow AND up) ) AND (TEXT.string: (Neurology~3 AND "kidney failure"~15) )
```





