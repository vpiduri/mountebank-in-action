# Chapter 7: Understanding behaviors

Any behavior that allows script execution (JavaScript or shell) requires starting mountebank
with the `--allowInjection` flag. To keep it secure, we'll also test with the `--localOnly` flag.

Since it's not the focus of the chapter, we'll ignore predicates for the examples.

## Listing 7.x: Using an `inject` response to add a dynamic timestamp

````
mb --allowInjection --localOnly --configfile examples/inject-response-timestamp.json &

# Should respond with current timestamp in response body
curl -i http://localhost:3000/

mb stop
````

## Listing 7.x: Combining an `is` response with a `decorate` behavior

````
mb --allowInjection --localOnly --configfile examples/decorate-timestamp.json &

# Should respond with current timestamp in response body
curl -i http://localhost:3000/

mb stop
````

## Listing 7.x: Adding a `decorate` behavior to recorded responses

````
mb --allowInjection --localOnly --configfile examples/proxy-decorate.json &

# From the server; x-rate-limit-remaining header = 50
curl -i http://localhost:3000/

# From recorded response: x-rate-limit-remaining-header = 25
curl -i http://localhost:3000/

# From recorded response: x-rate-limit-remaining-header = 0
curl -i http://localhost:3000/

# From recorded response: 429 status code
curl -i http://localhost:3000/

# The script stores state in this file
rm rate-limit.txt

mb stop
````

## Listing 7.x: Adding middleware through the `shellTransform` behavior

````
# installs the necessary ruby gems
bundle install

mb --allowInjection --localOnly --configfile examples/shell-transform.json &

# x-rate-limit-remaining header = 25
# Also has the current timestamp in body
curl -i http://localhost:3000/

# x-rate-limit-remaining-header = 0, with updated timestamp in body
curl -i http://localhost:3000/

# 429 status code
# Still has timestamp, since applyTimestamp.rb works independently
curl -i http://localhost:3000/

# The script stores state in this file
rm rate-limit.txt

mb stop
````

## Listing 7.x: Adding latency with a `wait` behavior

````
mb --configfile examples/wait.json &

# Returns in slightly over 3 seconds
time curl -i http://localhost:3000/

mb stop
````

## Listing 7.x: Adding a `repeat` behavior

````
mb --configfile examples/repeat.json &

# Returns 9999
curl -i http://localhost:3000/

# Returns 9999
curl -i http://localhost:3000/

# Returns 0
curl -i http://localhost:3000/

# Returns 9999
curl -i http://localhost:3000/

# Returns 9999
curl -i http://localhost:3000/

# Returns 0
curl -i http://localhost:3000/

mb stop
````

## Listing 7.x: Copying a value from the URL to the response body

````
mb --configfile examples/copy-regex.json &

# Returns 8731 in the response body
curl -i http://localhost:3000/accounts/8731

mb stop
````

## Listing 7.x: Copying a value from the URL to the response body using a grouped match

````
mb --configfile examples/copy-regex-group.json &

# Returns 5ea4d2b5 in the response body
curl -i http://localhost:3000/accounts/5ea4d2b5/profile

mb stop
````

## Listing 7.x: Copying a value using xpath

````
mb --configfile examples/xpath.json &

# Returns 5ea4d2b5 in the response body
curl -i -X POST http://localhost:3000/ --data'
<preferences>
  
</preferences>
'

mb stop
````