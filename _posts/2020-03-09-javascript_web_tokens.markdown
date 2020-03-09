---
layout: post
title:      "Javascript Web Tokens"
date:       2020-03-09 14:15:59 +0000
permalink:  javascript_web_tokens
---


So I finally got to my final project and discovered a concept that sped up my development time.  JSON Web Tokens (JWT for short).  As I was coding, I found I would have to login each time to view any changes.  This was because I had added user authentication to the routes.  As a result, the additional step to login  each re-render added unnecessary time.  Instead of removing the user authentication, I figured I was missing a solution and went hunting for an answer.  Turns out, JSON Web Tokens did the trick!

What are web tokens? JSON Web Token (JWT) is an easy way to secure an API. When a user authenticates first on a server, using for instance a standard login form, the server creates a token. This token includes some personal data, such as username or email address (https://www.jonathan-petitcolas.com/2014/11/27/creating-json-web-token-in-javascript.html).  More information on JWT can be found at https://jwt.io/introduction/.  

In a nutshell, when a client and server communicate, a web token can be transmitted and then decoded by the server.  If the credentials are authenticated, then the server will honor the request.  By adding this functionality, I was able to 'skip over' the login step because the authentication was happening behind the scenes.  If using Rails, you can use 

`gem install jwt`

to your project. Your Application Controller can look like this

```
class ApplicationController < ActionController::API 
    before_action :authenticate
    attr_accessor :current_user

    ALGORITHM = 'HS256'

        def encode(payload)
            JWT.encode(payload, auth_secret, ALGORITHM)
        end

        def decode(token)
            body = JWT.decode(token, auth_secret, true, { algorithm: ALGORITHM })[0]
            HashWithIndifferentAccess.new body
        rescue
            nil
        end
    

        def auth_secret
            ENV['AUTH_SECRET'] || Rails.application.secrets.secret_key_base
        end
    

        def logged_in?
            set_current_user
            !!@current_user
        end 

        def current_user
            @current_user 
        end

        def decoded_token
            if auth_header
              token = auth_header
              begin
                JWT.decode(token, auth_secret, true, algorithm: ALGORITHM) 
              rescue JWT::DecodeError
                nil
              end
            end
        end
    
        def set_current_user
            if has_valid_auth_type?
                user = User.find(decoded_token[0]['user_id'])
                if user 
                    @current_user = user 
                end 
            end 
        end 

        def authenticate
            unless logged_in?
                render json: {
                    success: false,
                    error: 'Invalid credentials'
                }, status: :unauthorized
            end 
        end 

    private 

        def auth_header 
            request.headers['Authorization'].to_s.scan(/Bearer (.*)$/).flatten.last
        end 

        def has_valid_auth_type?
            !!request.headers['Authorization'].to_s.scan(/Bearer/).flatten.first
        end 

    
end
```
