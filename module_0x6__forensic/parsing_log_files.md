# Parsing Log files


## Apache log file
Let's first list the important information we may need fro apache logs 

- [x] IP address
- [x] Time stamp 
- [x] HTTP method 
- [x] URI path
- [x] Response code
- [x] User agent 

To read a log file, I prefer to read it as lines 

```ruby
apache_logs = File.readlines "/var/log/apache2/access.log"
```

I was looking for a simple regex for apache logs. I found one [here](http://stackoverflow.com/questions/4846394/how-to-efficiently-parse-large-text-files-in-ruby).

```ruby
apache_regex = /(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}) - (.{0})- \[([^\]]+?)\] "(GET|POST|PUT|DELETE) ([^\s]+?) (HTTP\/1\.1)" (\d+) (\d+) "-" "(.*)"/
```

So I came up with this small method which parses and converts apache "access.log" file to an array contains list of hashes with our needed information.

```ruby
#!/usr/bin/evn ruby
# KING SABRI | @KINGSABRI

apache_logs = File.readlines "/var/log/apache2/access.log"

def parse(apache_logs) 
  apache_regex = /(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}) - (.{0})- \[([^\]]+?)\] "(GET|POST|PUT|DELETE) ([^\s]+?) (HTTP\/1\.1)" (\d+) (\d+) "-" "(.*)"/
  result_parse = []
  apache_logs.each do |log|
    parser = log.scan(apache_regex)[0]
    
    # If can't parse the log line for any reason.
    if log.scan(apache_regex)[0].nil?
      puts "Can't parse: #{log}\n\n"
      next
    end
    
    parse = 
        {
          :ip         => parser[0],
          :time       => parser[1],
          :method     => parser[3],
          :uri_path   => parser[4],
          :protocol   => parser[5],
          :code       => parser[6],
          :user_agent => parser[8]
        }
    result_parse << parse
  end
  
  return result_parse
end 

require 'pp'
pp parse(apache_logs)
```

Returns 
```
Can't parse: 127.0.0.1 - - [12/Dec/2015:20:09:05 +0300] "GET /icons/ubuntu-logo.png HTTP/1.1" 200 3689 "http://localhost/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36"


Can't parse: 127.0.0.1 - - [12/Dec/2015:20:09:05 +0300] "GET /favicon.ico HTTP/1.1" 404 500 "http://localhost/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36"


[{:ip=>"127.0.0.1",
  :time=>"",
  :method=>"GET",
  :uri_path=>"/",
  :protocol=>"HTTP/1.1",
  :code=>"200",
  :user_agent=>
   "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36"},
 {:ip=>"127.0.0.1",
  :time=>"",
  :method=>"GET",
  :uri_path=>"/etc/passwd",
  :protocol=>"HTTP/1.1",
  :code=>"404",
  :user_agent=>
   "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36"},
 {:ip=>"127.0.0.1",
  :time=>"",
  :method=>"GET",
  :uri_path=>"/etc/passwd",
  :protocol=>"HTTP/1.1",
  :code=>"404",
  :user_agent=>
   "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36"}]
```
