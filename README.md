## What is the play-pac4j library? [![Build Status](https://travis-ci.org/pac4j/play-pac4j.png?branch=master)](https://travis-ci.org/pac4j/play-pac4j)

The `play-pac4j` project is an authentication/authorization security library for Play framework. It's available under the Apache 2 license and based on the [pac4j](https://github.com/pac4j/pac4j) library.

Several libraries are available for the different versions of the Play framework and for the different languages:

| Play framework | Java library             | Scala library
|----------------|--------------------------|-----------------------------
| Play 2.0       | [play-pac4j_java v1.1.x](https://github.com/pac4j/play-pac4j/tree/1.1.x)   | [play-pac4j_scala2.9 v1.1.x](https://github.com/pac4j/play-pac4j/tree/1.1.x)
| Play 2.1       | [play-pac4j_java v1.1.x](https://github.com/pac4j/play-pac4j/tree/1.1.x)   | [play-pac4j_scala2.10 v1.1.x](https://github.com/pac4j/play-pac4j/tree/1.1.x)
| Play 2.2       | [play-pac4j_java v1.2.x](https://github.com/pac4j/play-pac4j/tree/1.2.x)   | [play-pac4j_scala v1.2.x](https://github.com/pac4j/play-pac4j/tree/1.2.x)
| Play 2.3       | [play-pac4j_java v1.4.x](https://github.com/pac4j/play-pac4j/tree/1.4.x)   | [play-pac4j_scala2.10](https://github.com/pac4j/play-pac4j/tree/1.4.x) and [play-pac4j_scala2.11 v1.4.x](https://github.com/pac4j/play-pac4j/tree/1.4.x)
| Play 2.4       | play-pac4j-java v2.0.x   | play-pac4j-scala_2.11 v2.0.x

It supports stateful and stateless [authentication flows](https://github.com/pac4j/pac4j/wiki/Authentication-flows) using external identity providers or direct internal credentials authenticator and user profile creator:

1. **OAuth** (1.0 & 2.0): Facebook, Twitter, Google, Yahoo, LinkedIn, Github... using the `pac4j-oauth` module
2. **CAS** (1.0, 2.0, SAML, logout & proxy) + REST API support using the `pac4j-cas` module
3. **HTTP** (form, basic auth, IP, header, GET/POST parameter authentications) using the `pac4j-http` module
4. **OpenID** using the `pac4j-openid` module
5. **SAML** (2.0) using the `pac4j-saml` module
6. **Google App Engine** UserService using the `pac4j-gae` module
7. **OpenID Connect** 1.0 using the `pac4j-oidc` module
8. **JWT** using the `pac4j-jwt` module
9. **LDAP** using the `pac4j-ldap` module
10. **relational DB** using the `pac4j-sql` module
11. **MongoDB** using the `pac4j-mongo` module.

See [all authentication mechanisms](https://github.com/pac4j/pac4j/wiki/Clients).


## Technical description

This project has **12 classes**:

1. the `PlayLogoutHandler` is the specific handler to support CAS logout
2. the `HttpActionHandler` interface and its `DefaultHttpActionHandler` implementation manage client HTTP actions in Play
3. the `AbstractConfigAction` is an abstract action to manage configuration
4. the `RequiresAuthentication` annotation is an action to be used to protect a resource
5. the `RequiresAuthenticationAction` handles the resource protection defined by the previous annotation
6. the `UserProfileController` is a controller to get the user profile
7. the `DataStore` interface and its `CacheStore` implementation manages the data to be saved in the store
8. the `ApplicationLogoutController` class handles the logout process
9. the `CallbackController` class is used to finish the authentication process
10. the `PlayWebContext` is the specific web context for Play

and is based on the `pac4j-core` library. Learn more by browsing the [play-pac4j Javadoc](http://www.pac4j.org/apidocs/play-pac4j/index.html) and the [pac4j Javadoc](http://www.pac4j.org/apidocs/pac4j/index.html).


## How to use it?

### Add the required dependencies (`play-pac4j-java` or `play-pac4j-scala_2.11` + `pac4j-*` libraries)

You need to add a dependency on the:

- `play-pac4j-java` library (<em>groupId</em>: **org.pac4j**, *latest version*: **2.0.0-SNAPSHOT**) if you code in Java
- `play-pac4j-scala_2.11` library (<em>groupId</em>: **org.pac4j**, *latest version*: **2.0.0-SNAPSHOT**) if you use Scala

as well as on the appropriate `pac4j` modules (<em>groupId</em>: **org.pac4j**, *version*: **1.8.0-SNAPSHOT**): the `pac4j-oauth` dependency for OAuth support, the `pac4j-cas` dependency for CAS support, the `pac4j-ldap` module for LDAP authentication, ...  

As snapshot dependencies are only available in the [Sonatype snapshots repository](https://oss.sonatype.org/content/repositories/snapshots/org/pac4j/), this repository must be added in the `resolvers` of your `build.sbt` file:

    resolvers ++= Seq( Resolver.mavenLocal,
        "Sonatype snapshots repository" at "https://oss.sonatype.org/content/repositories/snapshots/",
        "Pablo repo" at "https://raw.github.com/fernandezpablo85/scribe-java/mvn-repo/")


### Define the configuration (`Config` + `Clients` + `XXXClient` + `Authorizer`s)

Each authentication mechanism (Facebook, Twitter, a CAS server...) is defined by a client (implementing the `org.pac4j.core.client.Client` interface). All clients must be gathered in a `org.pac4j.core.client.Clients` class.  
All `Clients` must be defined in a `org.pac4j.core.config.Config` object which is itself bound for injection in a `SecurityModule`(or whatever you call it).

#### In Java:

    public class SecurityModule extends AbstractModule {
    
        ...
        
        @Override
        protected void configure() {
            FacebookClient facebookClient = new FacebookClient("fbId", "fbSecret");
            TwitterClient twitterClient = new TwitterClient("twId", "twSecret");

            FormClient formClient = new FormClient(baseUrl + "/theForm",
                    new SimpleTestUsernamePasswordAuthenticator(), new SimpleTestUsernameProfileCreator());
            IndirectBasicAuthClient basicAuthClient = new IndirectBasicAuthClient(new SimpleTestUsernamePasswordAuthenticator(),
                    new SimpleTestUsernameProfileCreator());
    
            CasClient casClient = new CasClient();
            casClient.setCasLoginUrl("http://mycasserver/login");
    
            SAML2ClientConfiguration cfg = new SAML2ClientConfiguration("resource:samlKeystore.jks",
                    "pac4j-demo-passwd", "pac4j-demo-passwd", "resource:openidp-feide.xml");
            cfg.setMaximumAuthenticationLifetime(3600);
            cfg.setServiceProviderEntityId("urn:mace:saml:pac4j.org");
            cfg.setServiceProviderMetadataPath(new File("target", "sp-metadata.xml").getAbsolutePath());
            final SAML2Client saml2Client = new SAML2Client(cfg);
    
            OidcClient oidcClient = new OidcClient();
            oidcClient.setClientID("id");
            oidcClient.setSecret("secret");
            oidcClient.setDiscoveryURI("https://accounts.google.com/.well-known/openid-configuration");
            oidcClient.addCustomParam("prompt", "consent");
    
            ParameterClient parameterClient = new ParameterClient("token", new JwtAuthenticator("salt"), new AuthenticatorProfileCreator<>());
    
            Clients clients = new Clients("http://localhost:8080/callback", facebookClient, twitterClient, formClient,
                    basicAuthClient, casClient, saml2Client, oidcClient, parameterClient);
    
            Config config = new Config();
            config.setClients(clients);
            config.getAuthorizers().put("customAuthorizer", new CustomAuthorizer());
            bind(Config.class).toInstance(config);
        }
    }

#### In Scala:

    class SecurityModule(environment: Environment, configuration: Configuration) extends AbstractModule {
    
      override def configure(): Unit = {
    
        val facebookClient = new FacebookClient("fbId", "fbSecret")
        val twitterClient = new TwitterClient("twId", "twSecret")

        val formClient = new FormClient(baseUrl + "/theForm",
          new SimpleTestUsernamePasswordAuthenticator(), new SimpleTestUsernameProfileCreator())
        val basicAuthClient = new IndirectBasicAuthClient(new SimpleTestUsernamePasswordAuthenticator(), new SimpleTestUsernameProfileCreator())
    
        val casClient = new CasClient()
        casClient.setCasLoginUrl("http://mycasserver/login")
    
        val cfg = new SAML2ClientConfiguration("resource:samlKeystore.jks", "pac4j-demo-passwd", "pac4j-demo-passwd", "resource:openidp-feide.xml")
        cfg.setMaximumAuthenticationLifetime(3600)
        cfg.setServiceProviderEntityId("urn:mace:saml:pac4j.org")
        cfg.setServiceProviderMetadataPath(new File("target", "sp-metadata.xml").getAbsolutePath())
        val saml2Client = new SAML2Client(cfg)
    
        val oidcClient = new OidcClient()
        oidcClient.setClientID("id")
        oidcClient.setSecret("secret")
        oidcClient.setDiscoveryURI("https://accounts.google.com/.well-known/openid-configuration")
        oidcClient.addCustomParam("prompt", "consent")
    
        val parameterClient = new ParameterClient("token", new JwtAuthenticator("salt"), new AuthenticatorProfileCreator[HttpCredentials, UserProfile])
    
        val clients = new Clients("http://localhost:8080/callback", facebookClient, twitterClient, formClient,
          basicAuthClient, casClient, saml2Client, oidcClient, parameterClient)
    
        val config = new Config()
        config.setClients(clients)
        bind(classOf[Config]).toInstance(config)
      }
    }

"http://localhost:8080/callback" is the url of the callback endpoint (see below). It may not be defined for REST support only.

You must notice that no authorizer is defined in Scala in the `Config` object as an `Authorizer` can be passed to the `RequiresAuthentication` function while only an `authorizerName` (`String`) can be passed to the `RequiresAuthentication` annotation (and thus requires to find the right `Authorizer` in the configuration).


### Define the data store (`CacheStore`)

Some of the data used by `play-pac4j` (user profile, tokens...) must be saved somewhere. Thus, a datastore must be defined in the `SecurityModule`.  
The only existing implementation is currently the `CacheStore` (where all data are saved into the `Cache`). <font color="red">If you have multiple Play nodes, you need a shared `Cache` between all your nodes.</font>

#### In Java:

    bind(DataStore.class).to(CacheStore.class);

#### In Scala:

    bind(classOf[DataStore]).to(classOf[CacheStore])


### Define the HTTP action handler (`DefaultHttpActionHandler`)

To handle specific HTTP actions (like redirections, forbidden / unauthorized pages), you need to define the appropriate `HttpActionHandler`. The only available implementation is currently the `DefaultHttpActionHandler`, but you can subclass it to define your own HTTP 401 / 403 error pages for example.
Its binding must be defined in the `SecurityModule`.

#### In Java:

    bind(HttpActionHandler.class).to(DefaultHttpActionHandler.class);

#### In Scala:

    bind(classOf[HttpActionHandler]).to(classOf[DefaultHttpActionHandler])


### Define the callback endpoint (only for stateful authentication mechanisms)

Some authentication mechanisms rely on external identity providers (like Facebook) and thus require to define a callback endpoint where the user will be redirected after login at the identity provider. For REST support only, this callback endpoint is not necessary.  
It must be defined in the `routes` file:

    GET    /callback   org.pac4j.play.CallbackController.callback()

You can also configure it by defining an instance in the `SecurityModule`.

#### In Java:

    CallbackController callbackController = new CallbackController();
    callbackController.setDefaultUrl("/");
    bind(CallbackController.class).toInstance(callbackController);

#### In Scala:

    val callbackController = new CallbackController()
    callbackController.setDefaultUrl("/")
    bind(classOf[CallbackController]).toInstance(callbackController)

And using it in the `routes` file:

    GET    /callback   @org.pac4j.play.CallbackController.callback()

The `defaultUrl` parameter defines where the user will be redirected after login if no url was originally requested.


### Protect an url (authentication + authorization)

You can protect an url and require the user to be authenticated by a client (and optionally have the appropriate roles / permissions) by using the `RequiresAuthentication` annotation or function.

#### In Java:

    @RequiresAuthentication(clientName = "FacebookClient")
    public Result facebookIndex() {
        return protectedIndex();
    }

The following parameters can be defined:

- `clientName`: the chosen authentication mechanism (like `FacebookClient` for example)
- `authorizerName` (optional): the authorizer name which will protect the resource (must exist in the authorizers configuration)
- `requireAnyRole` (optional): if one of the provided roles is necessary to access the resource
- `requireAllRoles` (optional): if all roles are necessary
- `isAjax` (optional): if this url is called in an AJAX way
- `allowDynamicClientSelection` (optional): if other clients can be used on this url (providing a *client_name* parameter in the url)
- `useSessionForDirectClient` (optional): if the session must be used (for REST client).

#### In Scala:

    def facebookIndex = RequiresAuthentication("FacebookClient") { profile =>
      Action { request =>
        Ok(views.html.protectedIndex(profile))
      }
    }

This function is available by using the `org.pac4j.play.scala.Security` trait. You must notice that the user profile is returned along the `RequiresAuthentication` function.

The following functions are available:

- `RequiresAuthentication[A](clientName: String)`
- `RequiresAuthentication[A](clientName: String, authorizer: Authorizer[P])`
- `RequiresAuthentication[A](clientName: String, requireAnyRole: String, requireAllRoles: String)`
- `RequiresAuthentication[A](clientName: String, authorizer: Authorizer[P], requireAnyRole: String, requireAllRoles: String, isAjax: Boolean, useSessionForDirectClient: Boolean, allowDynamicClientSelection: Boolean)`
- `RequiresAuthentication[A](parser: BodyParser[A], clientName: String, authorizer: Authorizer[P], requireAnyRole: String, requireAllRoles: String, isAjax: Boolean, useSessionForDirectClient: Boolean, allowDynamicClientSelection: Boolean)`

where the `authorizer` parameter is an `Authorizer` object and the `parser` a body parser.

Define the appropriate `org.pac4j.core.authorization.AuthorizationGenerator` and attach it to the client (using the `addAuthorizationGenerator` method) to compute the roles / permissions of the authenticated user.


### Get redirection urls

You can also explicitly compute a redirection url to a provider by using the `getRedirectAction` method, in order to create an explicit link for login. For example with Facebook:

#### In Java:

Either you inject the `Config` and `DataStore` in your controller or inherit from the `org.pac4j.play.java.UserProfileController`:

    public Result index() throws Exception {
        Clients clients = config.getClients();
        PlayWebContext context = new PlayWebContext(ctx(), dataStore);
        String urlFacebook = ((FacebookClient) clients.findClient("FacebookClient")).getRedirectAction(context, false, false).getLocation();
        return ok(views.html.index.render(urlFacebook));
    }

#### In Scala:

You need to use the `Security` trait:

    def index = Action { request =>
      val newSession = getOrCreateSessionId(request)
      val webContext = new PlayWebContext(request, dataStore)
      val clients = config.getClients()
      val urlFacebook = (clients.findClient("FacebookClient").asInstanceOf[FacebookClient]).getRedirectAction(webContext, false, false).getLocation;
      Ok(views.html.index(urlFacebook)).withSession(newSession)
    }

Notice you need to explictly call the `getOrCreateSessionId()` to force the initialization of the data store and attach the returned session to your result.


### Get the user profile

#### In Java:
 
You need to inherit from the `UserProfileController` and call the `getUserProfile()` method.
 
#### In Scala:

You need to extend from the `Security` trait and call the `getUserProfile(request: RequestHeader)` function. 

You can also use directly the `ProfileManager.get(true)` method (`false` not to use the session, but only the current HTTP request) and the `ProfileManager.isAuthenticated()` method. 

The retrieved profile is at least a `CommonProfile`, from which you can retrieve the most common properties that all profiles share. But you can also cast the user profile to the appropriate profile according to the provider used for authentication. For example, after a Facebook authentication:
 
    FacebookProfile facebookProfile = (FacebookProfile) commonProfile;


### Logout

You can log out the current authenticated user using the `ApplicationLogoutController` defined in the `routes` file and by calling the logout url ("/logout"):

    GET  /logout  org.pac4j.play.ApplicationLogoutController.logout()

A blank page is displayed by default unless an *url* parameter is provided. In that case, the user will be redirected to this specified url (if it matches the logout url pattern defined) or to the default logout url otherwise.

You can configure this controller by defining an instance in the `SecurityModule`.

#### In Java:

    ApplicationLogoutController logoutController = new ApplicationLogoutController();
    logoutController.setDefaultUrl("/");
    bind(ApplicationLogoutController.class).toInstance(logoutController);

#### In Scala:

    val logoutController = new ApplicationLogoutController()
    logoutController.setDefaultUrl("/")
    bind(classOf[ApplicationLogoutController]).toInstance(logoutController)

And using it in the `routes` file:

    GET  /logout  @org.pac4j.play.ApplicationLogoutController.logout()

The following parameters can be defined:

- `defaultUrl`: the default logout url if the provided *url* parameter does not match the `logoutUrlPattern`
- `logoutUrlPattern`: the logout url pattern that the logout url must match (it's a security check, only relative urls are allowed by default).


## Migration guide

`play-pac4j v2.0` is a huge refactoring of the previous version 1.5. It takes advantage of the new features of `pac4j` v1.8 (REST support, authorizations, configuration objects...)

It is fully based on dependency injection -> see [Play 2.4 migration guide](https://www.playframework.com/documentation/2.4.x/Migration24).

The `SecurityJavaController` and `JavaController` have been removed. You now need to use the `UserProfileController` in Java to get the user profile (you can also use the `ProfileManager` object directly).

The "target url" concept has disappeared as it was too complicated, it could be simulated though.

The `SecurityCallbackController` has been renamed as `CallbackController` (its original name) and the logout support has been moved to the `ApplicationLogoutController`.

The `JavaWebContext` and `ScalaWebContext` have been merged into a new `PlayWebContext`.

The `StorageHelper` has been removed, replaced by the `CacheStore` implementation where you can set the timeouts.

The static specific `Config` has been replaced by the default `org.pac4j.core.config.Config` object to define the `Clients` and the `Authorizer`s (for Java only).
  
Custom 401 / 403 HTTP error pages must be now defined by overriding the `DefaultHttpActionHandler`.


## Demo

The [play-pac4j-java-demo](https://github.com/pac4j/play-pac4j-java-demo) & [play-pac4j-scala-demo](https://github.com/pac4j/play-pac4j-scala-demo) are available with various authentication mechanisms: Facebook, Twitter, CAS, form, basic auth...


## Release notes

See the [release notes](https://github.com/pac4j/play-pac4j/wiki/Release-notes).


## Need help?

If you have any question, please use the following mailing lists:

- [pac4j users](https://groups.google.com/forum/?hl=en#!forum/pac4j-users)
- [pac4j developers](https://groups.google.com/forum/?hl=en#!forum/pac4j-dev)
