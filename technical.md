

INTRO
=====
The purpose of this document/page is to provide a technical analysis of the web2py PWA framework. It should provide a clear view of the general and specific w2p (web2py) technical mechanisms to build the PWA framwework. It is based on the [requirements _(only available locally)_](./requirements.md) document

## Audience
It is intended for (web) developpers and generally for developers and programmers.

## Full blown code generation technical analysis
**THE TECHNICAL ANALYSIS WILL BE A FULL BLOWN ANALYSIS, DOWN TO THE LAST FUNCTION AND PROCESS**
The code will be generated from that analysis and only the most inner processes hand-coded.


CONVENTIONS & PRINCIPLES
========================
## PWA display logic
In order to take full advantage of PWA logic, full html page should always be displayed first, before dynamic elements are called *if app is connected*
1.  Html page full display
2.  Dynamic elements calls & display

## General principles
1.  Repetition of API function with different names/paths is **good**, makes API functions easier to find...
    Ex:   `Site.database.page.fetch_components ==  Site.database.components.fetch`
2.  All API functions return value, { status:1|0|-1, message:'', code:200|404|500 }
3.  User only sees what he/she has access to
4.  Every access if checked against acl (security redundancy with previous)
5.  Callbacks: either [<function>, <{ parameters }>], or if object, assumes:
    i.  Site.pwa.display <function call> with given object
    ii  ...
6.  Objects types|parameters: complex object is either presented and thus handled accordingly in three possible formats:
    i.  By itself
    i.  In a list with it as first element and next elements as its parameters in correct order, or next element as a list of its parameter in correct order, or a dict of parameters
    ii. In a dict...
    **... this implies several levels of verification when receiving an object type as parameter, or parameter list/dict**
7.  Display: for visible transition (percept): always allow for a <loader...> for .300ms



## WYGIWYD
**What You Get Is What You Display** (from ghost platform)
No variables management, no processing in views: only display actions
=> Intermediary files between controllers & views for display preparing/processing

## Data types
- Lists as params instead of single value, as much as possible (flexibility, scalability)



PROCESS - TECHNICAL CASES
=========================

## The process
The process is (re)defined as this:
1.  The technical analysis to define the API & processes
2.  An Inception phase to implement the requirements using chosen frameworks (vue.js, pyjamas|pyjs, react), not looking at the technical requirements/analysis (in order to do things the framework's way) for vue and react, and only eventually for pyjs...
3.  And the actual implementation phase, with the chosen framework (Test, Validate - Release, Publish)

**List of & process details for chosen frameworks**:
-   w/ vue.js
-   w/ pyjamas|pyjs
-   [transcrypt]\: once completed with pyjs, write one complete case with transcrypt
-   w/ react

With this process the different frameworks will/should use the same backend to implement the requirements...

### Dev Stack
The pertinence of the [dev stack] will be analyzed within and at the end of the process.
And at the end of or within the process, integration of/into the [slow framework][dev stack] should be analyzed and done.


## Technical Cases
### Modules & functionalities
-   spa, pwa
-   acl: by page, by component, for menu items display
-   contextual help
-   form overlay
-   components per page
-   template: header, banner, sidebar, footer
-   pwa|spa blog & blog edition


### App creator presentation, App creation/management website
Choose and implement a **bootstrapious** template...
**Features**:
- Obvious help button
  - top right col-lg+, top middle col-md-
  - on click: 
        {
            Site.comm.send({
                channels:['mail', <v++>'chat'], names:'help_requested',
                operators:['client_service'] \(client_service | technical_support | admin\),
                details:{
                    user_details: Site.global.user_details,
                    page_details: Site.global.page_details,
                }
            }),
        }


### Technical case: Common
1. Site.config
    .user (see [requirements config (webseite, shop)](./requirements-config.md))
    .system ... {
        .page.default (
            'landing'
        )
        .page.components.default (
            menu,  sidebar, footer, pages'
        )
    }

2. Site.common.\_function()
    \# All API functions return value, { status:1|0|-1, message:'', code:200|404|500 }
    \# function that all other functions will _inherit_ from...

3. Site.pwa.display
    <html:page:load | w2p:page:load>
    -   Site.pwa.display = function({ page:_page_, components:null | [ _names_ ], display:'std'(default)|overlay }) {
            Site.global.page_details = {
                name: [page]
            }
            #...
            if(not components){
                page_components = Site.database.page.fetch_components({
                    name:[.],
                    user_details: Site.global.user_details (acl...)
                })
             || page_components = Site.database.components.fetch({
                    page:[.],
                    user_details: Site.global.user_details (acl...)
                })
                #...
                components = Site.config.system.page.components.default {
                    menu: Site.components.fetch({ name:. }),
                    sidebar: Site.components.fetch({ name:. }),
                    footer: Site.components.fetch({ name:. }),
                }
                components += page_components
            }
            Site.pwa.display({ components:components })
            #...
            return [value], {...}
        }

    -   Site.pwa.display({ component:menu })
            <w2p:component:load>
            Site.pwa.display = function(...) {
                Site.components.fetch({ name:. }) = {
                    r = Site.database.components.fetch({
                        name:[.],
                        user_details: Site.global.user_details (acl...)
                    })
                    Site.view.render({ component:., object:r })
                }
                #...
                return [value], {...}
            }

4.  <site:action button="[name]" event="[event]" success="Site.display(message\|page\|component)">
    Site.pwa.on(
        <w2p:event:handling?>
        { 
            page:_page_, component:_component_, button:_name_, event:_event_,
            success:{ page:_success-page_ },
        },
        function(){
            ...
            return [value], {...}
        }
    )
    </site:action>

5.  Site.pwa.on.\_access(
        <w2p:access_control>
        { page:[name]('website_configuration'), component:[name] },
        function(){
            <site:security:check_acl page="." component="." params="Site.global.user_details">
            b = Site.security.check_acl(
                { page:., component:. },
                { user_details: Site.global.user_details }
            )
            #...
            # b: 1 - user authorized,  0 - user not authorized,  -1 - user not existing
            if(b==1){
                Site.pwa.display({ page:., component:. })
            } else if(b==0) {
                // should _never_ be used according to _General Principle 3_
                Site.pwa.display({ component:'no_access' })
            } else if(b==-1) {
                Site.pwa.display({ component:'login_register' })
            }
            </site:security:check_acl>
            return [value], {...}            
        }        
    )


### App creation
#### Technical case: 
_User access: all_

1. Site.pwa.display
    <html:p:l | w2p:c:l>
    ({ page:'landing', components:null | [ _names_ ] })

2.  Site.pwa.on(
        <w2p:e:h?>
        { 
            page:null, component:null, button:'create_free_app', action:'click',
            success:{ page:'website_configuration' },
        },
3.      Site.pwa.display({ components:['free_app_shopcart'] })
        <w2p:c:l>
        function(){
            <site:security:check_acl page="." component="." params="Site.global.user_details"/>
            # ...
            r = Site.pwa.io.prompt(Name (r), phone (r), email)
            (default)r.user_role = 'standard'
            #...
4.          <w2p:cookies>Site.cookies.user.save(r)
            Site.global.user_details = r
            #...
            return true
        }
    ).continue({ success: Site.pwa.display({ . }) })


### Login|Registration
#### Technical case: 
_User access: all_

1.  Site.pwa.display
    <html:p:l | w2p:p:l>
    ({ page:'authentication', referral:<...> }){
        Site.security.check_acl
        # ...
        if(!Site.global.user_details){
            c = <w2p:cookies>Site.cookies.user.get()
            if(c){
                if(Site.cookies.user.has_profile()){
                    b = <site:security:check_acl page="referral" component="" params="this.c"/>
                    if(b){
                        Site.pwa.display({ page:<referral> })
                    } else {
                        Site.pwa.display({ page:Site.config.system.page.default, component:[['no_access', <referral>] })
                    }
                } else {
                    Site.pwa.display
                        <html:p:l | w2p:p:l>
                        ({ page:'user_profile', referral:<...> })
                }
            } else {
                Site.pwa.display
                    <w2p:c:l>
                    # **register page to require complete profile data to avoid extra redirection to profile page**
                    ({ page:'authentication', components:['login_register'], referral:<...> })
            }
        }
    }



### Contact, Newsletter
#### Technical case: 
_User access: all_
Standard flow.

1.  Site.pwa.display
    <html:p:l | w2p:p:l>
    ({ component:'contact_newsletter', which:'contact' | 'newsletter', referral:<...> }){
        if(!Site.global.user_details){
            Site.global.user_details = <w2p:cookies>Site.cookies.user.get()
        }
        <site:form:process name="contact_newsletter" params="Site.global.user_details">
        Site.form.contact_newsletter.fill(Site.global.user_details)
        Site.form.current = Site.form.contact_newsletter
2.      Site.pwa.on(
            <w2p:e:h?>
            {
                page:null, component:null, button:'submit', action:'click',
                process:Site.form.current.submit,
            },
            {
                page:null, component:null, form:'current', action:'submit',
                process:Site.form.current.submit,
                callbacks: Site.form.current.callbacks
            },
            Site.form.current.submit()
        ).continue({
            success: .callbacks.success(),
            error: .callbacks.error(),
        })
        </site:form:process>
    }


### Blog
#### Technical case: 
_User access: all_
Standard flow.


1.  Site.pwa.display
    <html:p:l | w2p:p:l>
    ({ component:'blog', referral:<...> }){
        if(!Site.global.user_details){
            Site.global.user_details = <w2p:cookies>Site.cookies.user.get()
        }
        Site.pwa.blog.init('navigation', 'edition', ...)
    }

2.  Site.pwa.on(
        <w2p:e:h?>
        {
            page:null, component:null, button:'submit', action:'click',
            process:Site.form.current.submit,
        },
        {
            page:null, component:null, form:'current', action:'submit',
            process:Site.form.current.submit,
            callbacks: Site.form.current.callbacks
        },
        Site.form.current.submit()
    ).continue({
        success: .callbacks.success(),
        error: .callbacks.error(),
    })


### App creation page
#### Technical case: creating configuration
_User access: all, registered, profile complete, role=app_creator-user_
_( The configuration wizard is accessible anonymously, with the option of registering at the end of the process to secure data )_

1.  Site.pwa.display(
        <w2p:access_control>
        { page:[name]('website_configuration'), component:'configuration_wizard' },
        function(){
            <site:form:process  name="configuration_wizard" data="(crud!=c){ fetch && fill }">
2.              <site:form:process  name="configuration_wizard.website"
                                    params="(crud==c)Site.global.user_details?"                                    
                                    callbacks="(crud==c)success:next:configuration_wizard.shop"/>

3.              <site:form:process  name="configuration_wizard.shop"
                                    params="(crud==c)Site.global.user_details?"
                                    options="(crud==c)skip"
                                    callbacks="(crud==c)success:next:configuration_wizard.blog"/>

4.              <site:form:process  name="configuration_wizard.blog"
                                    params="(crud==c)Site.global.user_details?"
                                    callbacks="(crud==c)success:finish:(show:result-link + options:register)"/>
            </site:form:process>
        }
    )


#### Technical case: updating configuration
_User access: registered, profile complete, role=app_creator-user_

1.  Site.pwa.display(
        <w2p:access_control>
        { page:[name]('website_configuration'), component:'configuration_wizard' },
        function(){
            <site:form:process  name="configuration_wizard" data="(crud!=c){ fetch && fill }"/>
        }
    )


#### Technical case: theme
_User access: registered, profile complete, role=app_creator-user_
*membership: all*

1.  Site.pwa.display(
        <w2p:access_control>
        { page:[name]('website_configuration'), component:'configuration_wizard' },
        function(){
            <site:form:process  ...>
                <site:form:process  name="configuration_wizard.website" ...>
                    Site.pwa.display(
                        <w2p:access_control/>
                        { button:'view_themes_button' },
                        <site:action    button="view_themes" event="click"
                                        success="Site.display(component:view_themes_component)">
2.                          Site.pwa.display(
                                <w2p:access_control/>
                                { component:'view_themes_component', acl:true }{
                                    Site.pwa.display(
                                        <w2p:access_control/>
                                        { components:['theme_thumbnails_grid'], display:'overlay' }{
                                            <site:action    button="view_theme_*" event="click"
                                                            success="Site.display(component:view_theme_component, theme:.)">
3.                                              Site.pwa.display(
                                                    <w2p:access_control/>
                                                    { component:'view_theme_component', acl:true }{
                                                        function(){
                                                            Site.pwa.display(theme:., output:'static')
                                                        }
                                                        <site:action    button="preview_theme" .../>
                                                            ...
                                                        </site:action>
                                                        <site:action    button="Cancel" event="click"
                                                                        success="Site.process.back(nr:-1)"/>
                                                })
                                            </site:action>
                                            <site:action    button="preview_theme_*" event="click"
                                                            success="Site.pwa.display(component:preview_theme_component, theme:., display:overlay)">
..                                              Site.pwa.display(
                                                    <w2p:access_control/>
                                                    { component:'preview_theme_component', theme:., display:'overlay', acl:true }{
                                                        <site:action    button="apply_theme" event="click"
                                                                        success="Site.process.apply_theme(theme:.)">
                                                            Site.pwa.display(
                                                                <w2p:access_control/>
                                                                { button:'View website', display:'new_window' }
                                                        </site:action>
                                                        <site:action    button="Cancel" event="click"
                                                                        success="Site.process.back(nr:-1)"/>
                                                })
                                            </site:action>                                            
                                    })
                            })
                        </site:action>
                </site:form:process>
            </site:form:process>
        }
    )

#### Technical case: custom theming
v0.9.0
_User access: registered, profile complete, role=app_creator-user_
*membership: paid*


1.  Site.pwa.display(
        <w2p:access_control>
        { page:[name]('website_configuration'), component:'configuration_wizard' },
        function(){
            <site:form:process  ...>
                <site:form:process  name="configuration_wizard.website" ...>
                    Site.pwa.display(
                        <w2p:access_control/>
                        { button:'upload_theme' },
                        <site:action    button="upload_theme" event="click"
                                        success="Site.display(component:upload_theme_component)">
                            Site.pwa.display(
                                <w2p:access_control/>
                                { component:'upload_theme_component', acl:true }{
                                    <site:message   text="theming_overview+tutoriel"
                                                    link="page:theming_tutoriel, display:new_window"/>
                                    <site:action    button="upload_theme"  type="button" event="click"
                                                    success="Site.process.upload_theme(theme:this.file)">
2.                                      Site.process.upload_theme(
                                            <w2p:access_control/>
                                            { success: Site.pwa.display({ button:'preview_uploaded_theme' }) }
                                            function(){
                                                zipfile = this.file

                                                **with(Site.process.fstorage.user_theme_folder){**
                                                    userThemeF = .get_obj(user_details:Site.global.user_details)
                                                    if(!Site.process.fstorage.exists(userThemeF.dir)){
                                                        Site.process.fstorage.make(userThemeF.dir)
                                                        userThemeF.themes = []
                                                    }                                                
                                                    .make_new_theme(zipfile){
                                                        unique_theme_name = .make_theme_unique_name(zipfile)
                                                        .make_theme_folder(unique_theme_name)
                                                        .unzip_theme_folder(dst:unique_theme_name, zipfile){
                                                            /**folder list according to templating options and possibilities**/
                                                            _templates/...   (website & blog)
                                                            images/...      (website & blog)
                                                            css/...         (all:website, shop & blog)
                                                            js/...          (website & blog)_
                                                        }
                                                    }
                                                    b = Site.process.fstorage The system creates a new uuid current custom user theme folder
                                                    Site.process.file() The system creates a new uuid current custom user theme folder
                                                        ... and displays a "Preview" button
                                                        -   User clicks on "Preview"                                            
                                                **}**

                                                  return [unique_theme_name], ...
                                            }
                                            <site:message   text="theme_uploaded"/>
                                            <site:action    button="preview_theme" ...>
3.                                              ...
                                            </site:action>
                                            <site:action    button="apply_theme" event="click"
                                                            success="Site.process.apply_theme(theme:.)" ...>
                                                            ...
                                            </site:action>
                                        )
                                    </site:action>
                                    <site:action    button="close"  type="button" event="click"
                                                    success="Site.process.back(nr:-1)"/>
                            })
                        </site:action>
                        <site:action    button="list_uploaded_themes" event="click"
                                        success="Site.display(component:uploaded_themes_component, display:'overlay')">
                        </site:action>
                        <site:action    button="view_website" event="click"
                                        success="Site.pwa.display(page:Site.system.page.default, display:'new_window')">
                        </site:action>
                </site:form:process>
            </site:form:process>
        }
    )


[dev stack]: ~/@emelit/src/dev/dev-stack.md
[transcrypt]: http://transcrypt.org/