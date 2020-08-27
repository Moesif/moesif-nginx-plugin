# Moesif NGINX module

Native plugin for Nginx module to monitor, analyze API traffic in Moesif advanced analytics platform

## How it works?

This low overhead module captures API request/response headers and body and periodically posts to Moesif  servers. The module provides several options, explained below,
to be able to choose which API's data is captured and what part of API headers/body can be sent to Moesif.

## How to install

### 1. Install the Moesif module

Download pre-build binaries from [Releases](https://github.com/Moesif/moesif-nginx-plugin/tree/master/modules/),
place them into `./modules` sub-directory of `nginx`. 

Assuming standard NGINX plus deployment, create a symlink for the module as shown below.

```
sudo ln -s /etc/nginx/modules/ngx_moesif_http_filter_module_1.19.0.so /etc/nginx/modules/ngx_http_moesif_filter_module.so
```
### 2. Edit your `nginx.conf`

Add the following line at the beginning of `nginx.conf`
_If you're unsure of the installation path, you can find it via: `find / -name "moesif" -type d`.

```nginx
load_module modules/ngx_http_moesif_filter_module.so;
```

Also, edit your `nginx.conf` file to add the Moesif Plus plugin:

```nginx
http{
    moesif  on;
    moesif_application_id your-moesif-application-id
    ...
    server {
        server_name acmeinc.com;
        
        location /api {
            proxy_pass 127.0.0.1:8080
        }
    }
}
```

### 3. Reload NGINX

Reload the NGINX config with the command `nginx -s reload`

*Alternatively, you can install this module manually with the Nginx source, see the [installation instructions](#build-from-source)*


## Configuration

You can modify your configuration to install on specific services or to add additional data to the logged API calls such as User Id or Metadata.

Example:

```nginx
http{
    # turn on Moesif  function
    moesif  on;
    moesif_application_id  your-moesif-application-id;  # required field
    moesif_sync_interval 1;  # optional field to configure how often API data is sent, defaults to 2 seconds
    moesif_events_cache_size 50;  # optional field to configure events cache size, defaults to 25
    moesif_metadata type http;    # optional metadata
    ...
    server {
        server_name example.com;
        
        moesif_metadata server east-1; #optional metadata

        location /loc1 {
                #moesif data is captured since moesif is on at a global level
            moesif_user_id_header  X-USER-ID;
            moesif_company_id_header X-COMPANY_ID;
            moesif_api_version_header X-API-VERSION; 
            moesif_api_weight  1;
            ...    
            proxy_pass 127.0.0.1:8080
        }
        
        location /loc2 {
            moesif  off;  # moesif disabled. No API events captured at this location  
            ...
     
        }
        
        location /loc3 {
            moesif  on;
            moesif_skip_body on; # body is not captured at this location
            moesif_metadata api mgmt; #optional metadata, all the metadata above is inherited
            ...
     
        }
        
    }

}
```

# Directives

moesif_application_id
--------------------
**syntax:** *moesif_application_id \<application_id>*

**default:** *-*

**context:** *http*

Your Moesif Application Id can be found in the [_Moesif Portal_](https://www.moesif.com/).

You can always find your Moesif Application Id at any time by logging 
into the [_Moesif Portal_](https://www.moesif.com/), click on the top right menu,
and then clicking _Installation_.

moesif_events_cache_size
--------------------
**syntax:** *moesif_events_cache_size \<cache_size>*

**default:** *cache_size 25*

**context:** *http*

Sets the `cache_size` as unsigned number.
This is used to determine internal cache size module uses to save events.

moesif_sync_interval
------------------------
**syntax:** *moesif_sync_interval \<seconds>*

**default:** *moesif_sync_interval 2*

**context:** *http*

Specifies the interval events are synced to Moesif.  Defaults to 2 seconds.

moesif
--------------------
**syntax:** *moesif on | off*

**default:** *moesif off*

**context:** *http, server, location*

Enables/Disables sending API data to Moesif at Main, Server or particular location level. Location level configuration takes precedence to Server level configuration. Similarly Server level configuration takes precedence to Main level configuration

moesif_skip_body
--------------------
**syntax:** *moesif_skip_body on | off*

**default:** *off*

**context:** *http, server, location*

This flag can be used to skip body being sent to Moesif at location/server/global level

moesif_user_id_header
--------------------
**syntax:** *moesif_user_id_header \<header_name>*

**default:** **

**context:** *http, server, location*

if set, module extracts header_name from API headers and uses that as 'user' of the API request. header_name could 
be request or response header. Response headers are matched first and its case in sensitive

moesif_company_id_header
--------------------
**syntax:** *moesif_company_id_header \<header_name>*

**default:** **

**context:** *http, server, location*

if set, module extracts header_name from API headers and uses that as 'company' of the API request. header_name could 
be request or response header. Response headers are matched first and its case in sensitive

moesif_api_version_header
------------------------
**syntax:** *moesif_api_version \<api_version>*

**default:** **

**context:** *http, server, location*

if set, module extracts header_name from API headers and uses that as 'api_version' of the API request. header_name could 
be request or response header. Response headers are matched first and its case in sensitive

moesif_api_weight
--------------------
**syntax:** *moesif_api_weight \<weight>*

**default:** **

**context:** *http, server, location*

This directive can be used to set weight of API request at location/server/global level

moesif_metadata
--------------------
**syntax:** *moesif_metadata \<key> \<value>*

**default:** **

**context:** *http, server, location*

Additional metadata can be configured with this directive to send as part of API metadata. Metadata configured at higher levels is inherited

# Advanced customization

Its possible to customize this module further using the interfaces available in ngx_http_moesif_custom_configuration.c and then see the [installation instructions](#Installation) to build module.

ngx_http_moesif_skip_event(ngx_http_request_t *r)
-------------------------------------------------
if a particular event should be skipped logging, this function can be defined The parameter passed in is an ngx_http_request,
which contains request headers, response headers, request body.

ngx_http_moesif_identify_user(ngx_http_request_t *r, ngx_str_t *user)
-------------------------------------------------------------------------
user id can be set in user->data after extracting it from ngx_http_request_t

ngx_http_moesif_identify_company(ngx_http_request_t *r, ngx_str_t *company)
-----------------------------------------------------------------------------
company id can be set in company->data after extracting it from ngx_http_request_t

ngx_http_moesif_get_api_version(ngx_http_request_t *r, ngx_str_t *api_version)
-------------------------------------------------------------------------------
API version of equest can be set in api_version->data after extracting it from ngx_http_request_t

ngx_http_moesif_get_metadata(ngx_http_request_t *r, ngx_http_moesif_map_t *metadata, ngx_pool_t *pool)
--------------------------------------------------------------------------------------------------------
Additional metadata can be set in metadata. Any needed memory can be allocated from pool

ngx_http_moesif_mask_content(ngx_http_moesif_event_t *r)
-------------------------------------------------------
This function can be configured to mask content as desired before events sent to moesif. Moesif event model is defined 
in [ngx_http_moesif_event.h](https://github.com/Moesif/ngx_http_moesif_filter_module/blob/master/src/ngx_http_moesif_event.h)



---
## Build from source

### Step 1

There are several ways to integrate Moesif module into NGINX.

* Download pre-build binaries from [Releases](https://github.com/Moesif/ngx_http_moesif_filter_module/releases). Or

* Build the binaries from sources

```
# grab nginx source code from nginx.org, then cd to /path/to/nginx-src/
git clone https://github.com/Moesif/ngx_http_moesif_filter_module.git

# to build as `static` module
./configure --prefix=/opt/nginx --with-stream --add-module=ngx_http_moesif_filter_module
make && make install

# to build as `dynamic` module
./configure --prefix=/opt/nginx --add-dynamic-module=ngx_http_moesif_filter_module

make modules
```

### Step 2 (dynamic module only)

Add the following lines at the beginning of `nginx.conf`:

```
load_module modules/ngx_http_moesif_filter_module.so;

```

### Step 3

```
http {
  moesif                  on;
  moesif_application_id   your-application-id;
  ...
}
```
