---
title: Plug-in Basics
slug: Plugins/Guide/Plug-in_Basics
tags:
  - Gecko Plugin API Reference
  - Guide
  - NPAPI
  - Plugins
---

<h3 id="How_Plug-ins_Are_Used">How Plug-ins are used</h3>

<p>Plug-ins offer a rich variety of features that can increase the flexibility of <a href="/en-US/Gecko">Gecko</a>-based browsers. Plug-ins like these are now available:</p>

<ul>
 <li>multimedia viewers such as Adobe Flash and Adobe Acrobat</li>
 <li>utilities that provide object embedding and compression/decompression services</li>
 <li>applications that range from personal information managers to games</li>
</ul>

<p>The range of possibilities for using plug-in technology seems boundless, as shown by the growing numbers of independent software vendors who are creating new and innovative plug-ins.</p>

<p>With the Plug-in API, you can create dynamically loaded plug-ins that can:</p>

<ul>
 <li>register one or more MIME types</li>
 <li>draw into a part of a browser window</li>
 <li>receive keyboard and mouse events</li>
 <li>obtain data from the network using URLs</li>
 <li>post data to URLs</li>
 <li>add hyperlinks or hotspots that link to new URLs</li>
 <li>draw into sections on an <a href="/en-US/docs/Web/HTML">HTML</a> page</li>
 <li>communicate with <a href="/en-US/docs/Web/JavaScript">JavaScript</a>/<a href="/en-US/docs/Web/API/Document_Object_Model">DOM</a> from native code</li>
</ul>

<p>You can see which plug-ins are installed on your system and have been properly associated with the browser by consulting the Installed Plug-ins page. Go to the Help menu, and click Help and then About Plug-ins. Type "about:plugins" in the Location bar. The Installed Plug-ins page lists each installed plug-in along with its MIME type or types, description, file extensions, and the current state (enabled or disabled) of the plug-in for each MIME type assigned to it. Notice in view-source that this information is gathered from the JavaScript.</p>

<p>Because plug-ins are platform-specific, you must port them to every operating system and processor platform upon which you want to deploy your plug-in.</p>

<h4 id="Plug-ins_and_Helper_Applications">Plug-ins and helper applications</h4>

<p>Before plug-ins, there were helper applications. A helper application is a separate, free-standing application that can be started from the browser. Like a plug-in, the browser starts a helper application when the browser encounters a MIME type that is mapped to it. Unlike a plug-in, a helper application runs separately from the browser in its own application space and does not interact with the browser or the web.</p>

<p>When the browser encounters a MIME type, it always searches for a registered plug-in first. If there are no matches for the MIME type, it looks for a helper application.</p>

<p>Plug-ins and helper applications fill different application needs. For more information about helper applications, refer to the Netscape online help.</p>

<h3 id="How_Plug-ins_Work">How plug-ins work</h3>

<p>The life cycle of a plug-in, unlike that of an application, is completely controlled by the web page that calls it. This section gives you an overview of the way that plug-ins operate in the browser.</p>

<p>When Gecko starts, it looks for plugin modules in particular places on the system. For more information about where Gecko looks for plugin modules on different systems, see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#How_Gecko_Finds_Plug-ins">How Gecko Finds Plug-ins</a>.</p>

<p>When the user opens a page that contains embedded data of a media type that invokes a plug-in, the browser responds with the following sequence of actions:</p>

<ul>
 <li>check for a plug-in with a matching MIME type</li>
 <li>load the plug-in code into memory</li>
 <li>initialize the plug-in</li>
 <li>create a new instance of the plug-in</li>
</ul>

<p>Gecko can load multiple instances of the same plug-in on a single page, or in several open windows at the same time. If you are browsing a page that has several embedded RealAudio clips, for example, the browser will create as many instances of the RealPlayer plug-in as are needed (though of course playing several RealAudio files at the same time would seldom be a good idea). When the user leaves the page or closes the window, the plug-in instance is deleted. When the last instance of a plug-in is deleted, the plug-in code is unloaded from memory. A plug-in consumes no resources other than disk space when it is not loaded. The next section, <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Understanding_the_Runtime_Model">Understanding the Runtime Model</a>, describes these stages in more detail.</p>

<h3 id="Understanding_the_Runtime_Model">Understanding the runtime model</h3>

<p>Plug-ins are dynamic code modules that are associated with one or more MIME types. When the browser starts, it enumerates the available plug-ins (this step varies according to platform), reads resources from each plug-in file to determine the MIME types for that plug-in, and registers each plug-in library for its MIME types.</p>

<p>The following stages outline the life of a plug-in from loading to deletion:</p>

<ul>
 <li>When Gecko encounters data of a MIME type registered for a plug-in (either embedded in an HTML page or in a separate file), it dynamically loads the plug-in code into memory, if it hasn't been loaded already, and it creates a new instance of the plug-in.</li>
</ul>

<p>Gecko calls the plug-in API function <a href="/en-US/NP_Initialize">NP_Initialize</a> when the plug-in code is first loaded. By convention, all of the plug-in specific functions have the prefix "NPP", and all of the browser-specific functions have the prefix "NPN".</p>

<div class="note">
<p><strong>Note:</strong> <code>NP_Initialize</code> and <code>NP_Shutdown</code> are not technically a part of the function table that the plug-in hands to the browser. The browser calls them when the plug-in software is loaded and unloaded. These functions are exported from the plug-in DLL and accessed with a system table lookup, which means that they are not related to any particular plug-in instance. Again, see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Initialization_and_Destruction">Initialization and Destruction</a> for more information about initializing and destructing.</p>
</div>

<ul>
 <li>The browser calls the plug-in API function <a href="/en-US/NPP_New">NPP_New</a> when the instance is created. Multiple instances of the same plug-in can exist (a) if there are multiple embedded objects on a single page, or (b) if several browser windows are open and each displays the same data type.</li>
 <li>A plug-in instance is deleted when a user leaves the instance's page or closes its window; Gecko calls the function <a href="/en-US/NPP_Destroy">NPP_Destroy</a> to inform the plug-in that the instance is being deleted.</li>
 <li>When the last instance of a plug-in is deleted, the plug-in code is unloaded from memory. Gecko calls the function <a href="/en-US/NP_Shutdown">NP_Shutdown</a>. Plug-ins consume no resources (other than disk space) when not loaded.</li>
</ul>

<div class="note">
<p><strong>Note:</strong> Plug-in API calls and callbacks use the main Navigator thread. In general, if you want a plug-in to generate additional threads to handle processing at any stage in its lifespan, you should be careful to isolate these from Plug-in API calls.</p>
</div>

<p>See <a href="/en-US/docs/Gecko_Plugin_API_Reference/Initialization_and_Destruction">Initialization and Destruction</a> for more information about using these methods.</p>

<h3 id="Plug-in_Detection">Plug-in detection</h3>

<p>Gecko looks for plug-ins in various places and in a particular order. The next section, <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#How_Gecko_Finds_Plug-ins">How Gecko Finds Plug-ins</a>, describes these rules, and the following section, <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Checking_Plug-ins_by_MIME_Type">Checking Plug-ins by MIME Type</a>, describes how you can use JavaScript to locate plug-ins yourself and establish which ones are to be registered for which MIME types.</p>

<h4 id="How_Gecko_Finds_Plug-ins">How Gecko finds plug-ins</h4>

<p>When a Gecko-based browser starts up, it checks certain directories for plug-ins, in this order:</p>

<h5 id="Windows">Windows</h5>

<ul>
 <li>Directory pointed to by <code>MOZ_PLUGIN_PATH</code> environment variable.</li>
 <li><code>%APPDATA%\Mozilla\plugins</code>, where <code>%APPDATA%</code> denotes per-user <code>Application Data</code> directory.</li>
 <li><a href="/en-US/Shipping_a_plugin_as_a_Toolkit_bundle">Plug-ins within toolkit bundles</a>.</li>
 <li><code>Profile directory\plugins</code>, where <code>Profile directory</code> is a user profile directory.</li>
 <li>Directories pointed to by <code>HKEY_CURRENT_USER\Software\MozillaPlugins\*\Path</code> registry value, where <code>*</code> can be replaced by any name.</li>
 <li>Directories pointed to by <code>HKEY_LOCAL_MACHINE\Software\MozillaPlugins\*\Path</code> registry value, where <code>*</code> can be replaced by any name.</li>
</ul>

<h5 id="Mac_OS_X">Mac OS X</h5>

<ul>
 <li><code>~/Library/Internet Plug-Ins</code>.</li>
 <li><code>/Library/Internet Plug-Ins</code>.</li>
 <li><code>/System/Library/Frameworks/JavaVM.framework/Versions/Current/Resources</code>.</li>
 <li><a href="/en-US/Shipping_a_plugin_as_a_Toolkit_bundle">Plug-ins within toolkit bundles</a>.</li>
 <li><code>Profile directory/plugins</code>, where <code>Profile directory</code> is a user profile directory.</li>
</ul>

<h5 id="Linux">Linux</h5>

<ul>
 <li>Directory pointed to by <code>MOZ_PLUGIN_PATH</code> environment variable. For example:</li>
</ul>

<pre class="brush: bash">#!/bin/bash

export MOZ_PLUGIN_PATH=/usr/lib64/mozilla/plugins
exec /usr/lib64/firefox/firefox</pre>

<ul>
 <li><code><em>Profile directory</em>/plugins</code>, where <em><code>Profile directory</code></em> is the directory of the current user profile.</li>
 <li><code>~/.mozilla/plugins</code>.</li>
 <li><code>/usr/lib/mozilla/plugins</code> (the 64-bit Firefox checks <code>/usr/lib64/mozilla/plugins</code> as well).</li>
 <li><code>/usr/lib64/firefox/plugins</code> (for 64-bit Firefox)</li>
</ul>

<p><strong>Note:</strong> Firefox Nightly checks a subset of these locations. Also some Linux distributions provide a system browser configured differently.</p>

<blockquote>
<p><em>Advanced:</em> You can determine which directories a Gecko program checks with the Linux <code>strace</code> command, for example:</p>

<pre class="brush: bash">strace -e open /usr/bin/firefox 2&gt;&amp;1 | grep plugin</pre>

<p>But with version firefox-41.0.2 we can not check. I found other way how check which paths support Firefox :</p>

<pre class="brush: bash">$ strace -y /usr/bin/firefox 2&gt;&amp;1 | grep acces | grep -v search | grep plugins
access("/home/user_name/.mozilla/firefox/dqh2nb5k.default-1441864569209/plugins", F_OK) = -1 ENOENT (No such file or directory)
access("/home/user_name/.mozilla/plugins", F_OK) = -1 ENOENT (No such file or directory)
access("/usr/lib64/firefox/browser/plugins", F_OK) = -1 ENOENT (No such file or directory)
access("/usr/lib/mozilla/plugins", F_OK) = 0</pre>

<p>This output I have after close Firefox. I checked also this command with above script (with environment variable) on my system and also working.</p>

<p>However, primary working path with binary file was <code>/usr/lib/mozilla/plugins</code><br>
 for 32-bit and 64-bit Linux distributions and looks still working.<br>
 Firefox and OpenSuse probably use "<code>MOZ_PLUGIN_PATH</code> environment variable" in script to run Firefox, so in this way <code>/usr/lib64/mozilla/plugins also should be supported.</code></p>

<p>About distributions:<br>
 Example Debian 64bit probably use:<br>
 /lib/x86_64-linux-gnu/ --&gt; for 64 libs<br>
 /lib/i386-linux-gnu/ --&gt; for 32 libs<br>
 if exist<br>
 /lib32/ --&gt; this is symlinked (or bind mounted) desired proper directory<br>
 /lib64/ --&gt; this is symlinked (or bind mounted) desired proper directory<br>
 more in https://wiki.debian.org/Multiarch/TheCaseForMultiarch<br>
 if something wrong, please edit.</p>

<p>Example Fedora 64bit use:<br>
 /lib/ --&gt; for 32 bit libs<br>
 /lib64/ --&gt; for 64 bit libs</p>
</blockquote>

<ul>
 <li><a href="/en-US/Shipping_a_plugin_as_a_Toolkit_bundle">Plug-ins within toolkit bundles</a>.</li>
</ul>

<p>To find out which plug-ins are currently installed visit about:plugins. Gecko displays a page listing all installed plug-ins and the MIME types they handle, as well as optional descriptive information supplied by the plug-in.</p>

<p>On Windows, installed plug-ins are automatically configured to handle the MIME types that they support. If multiple plug-ins handle the same MIME type, the first plug-in registered handles the MIME type. For information about the way MIME types are assigned, see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Development_Overview#Registering_Plug-ins">Registering Plug-ins</a>.</p>

<h4 id="Checking_Plug-ins_by_MIME_Type">Checking plug-ins by MIME type</h4>

<p>The <code>enabledPlugin</code> property in JavaScript can be used to determine which plug-in is configured for a specific MIME type. Though plug-ins may support multiple MIME types and each MIME type may be supported by multiple plug-ins, only one plug-in can be configured for a MIME type. The <code>enabledPlugin</code> property is a reference to a Plugin object that represents the plug-in that is configured for the specified MIME type.</p>

<p>You might need to know which plug-in is configured for a MIME type, for example, to dynamically create an <code>object</code> element on the page if the user has a plug-in configured for the MIME type.</p>

<p>The following example uses JavaScript to determine whether the Adobe Flash plug-in is installed. If it is, a movie is displayed.</p>

<pre class="brush: js">// Can we display Adobe Flash movies?
var mimetype = navigator.mimeTypes["application/x-shockwave-flash"];

if (mimetype) {
   // Yes, so can we display with a plug-in?
   var plugin = mimetype.enabledPlugin;
   if (plugin) {
      // Yes, so show the data in-line
      document.writeln("Here\'s a movie: &lt;object data='mymovie.swf' height='100' width='100'&gt;&lt;/object&gt;");
   } else {
      // No, so provide a link to the data
      document.writeln("&lt;a href='mymovie.swf'&gt;Click here&lt;/a&gt; to see a movie.");
   }
} else {
   // No, so tell them so
   document.writeln("Sorry, can't show you this movie.");
}
</pre>

<h3 id="Overview_of_Plug-in_Structure">Overview of plug-in structure</h3>

<p>This section is an overview of basic information you will need as you develop plug-ins.</p>

<ul>
 <li><a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Understanding_the_Plug-in_API">Understanding the Plug-in API</a></li>
 <li><a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Plug-ins_and_Platform_Independence">Plug-ins and Platform Independence</a></li>
</ul>

<h4 id="Understanding_the_Plug-in_API">Understanding the plug-in API</h4>

<p>A plug-in is a native code library whose source conforms to standard C syntax. The Plug-in Application Programming Interface (API) is made up of two groups of functions and a set of shared data structures.</p>

<ul>
 <li>Plug-in methods are functions that you implement in the plug-in; Gecko calls these functions. The names of all the plug-in functions in the API begin with <code>NPP_</code>, for example, <code>NPP_New</code>. There are also a couple of functions (i.e., <code>NP_Initialize</code> and <code>NP_Shutdown</code>) that are direct library entry points and not related to any particular plug-in instance.</li>
 <li>Browser methods are functions implemented by Gecko; the plug-in calls these functions. The names of all the browser functions in the API begin with <code>NPN_</code>, for example, <code>NPN_Write</code>.</li>
 <li>Data structures are plug-in-specific types defined for use in the Plug-in API. The names of structures begin with <code>NP</code>, for example, <code>NPWindow</code>.</li>
</ul>

<p>All plug-in names in the API start with <code>NP</code>. In general, the operation of all API functions is the same on all platforms. Where this varies, the reference entry for the function in the reference section describes the difference.</p>

<h4 id="Plug-ins_and_Platform_Independence">Plug-ins and platform independence</h4>

<p>A plug-in is a dynamic code module that is native to the specific platform on which the browser is running. It is a code library, rather than an application or an applet, and runs only from the browser. Although plug-ins are platform-specific, the Plug-in API is designed to provide the maximum degree of flexibility and to be functionally consistent across all platforms. This guide notes platform-specific differences in coding for the MS Windows, Mac OS X, and Unix platforms.</p>

<p>You can use the Plug-in API to write plug-ins that are media type driven and provide high performance by taking advantage of native code. Plug-ins give you an opportunity to seamlessly integrate platform-dependent code and enhance the Gecko core functionality by providing support for new data types.</p>

<p>The plug-in file type depends on the platform:</p>

<ul>
 <li>MS Windows: .DLL (Dynamic Link Library) files</li>
 <li>Unix: .SO or .DSO (Shared Objects) files</li>
 <li>Mac OS X: PPC/x86/Universal loadable Mach-O bundle</li>
</ul>

<h3 id="Windowed_and_Windowless_Plug-ins">Windowed and windowless plug-ins</h3>

<p>You can write plug-ins that are drawn in their own native windows or frames on a web page. Alternatively, you can write plug-ins that do not require a window to draw into. Using windowless plug-ins extends the possibilities for web page design and functionality. Note, however, that plug-ins are windowed by default, because windowed plug-ins are in general easier to develop and more stable to use.</p>

<ul>
 <li>A windowed plug-in is drawn into its own native window on a web page. Windowed plug-ins are opaque and always come to the top HTML section of a web page.</li>
 <li>A windowless plug-in need not be drawn in a native window; it is drawn in its own drawing target. Windowless plug-ins can be opaque or transparent, and can be invoked in HTML sections.</li>
</ul>

<p>Whether a plug-in is windowed or windowless depends on how you define it.</p>

<p>The way plug-ins are displayed on the web page is determined by the HTML element that invokes them. This is up to the content developer or web page author. Depending on the element and its attributes, a plug-in can be visible or hidden, or can appear as part of a page or as a full page in its own window. A web page can display a windowed or windowless plug-in in any HTML display mode; however, the plug-in must be visible for its window type to be meaningful. For information about the way HTML determines plug-in display mode, see "Using HTML to Display Plug-ins."</p>

<h3 id="Using_HTML_to_Display_Plug-ins">Using HTML to Display Plug-Ins</h3>

<p>When a user browses to a web page that invokes a plug-in, how the plug-in appears (or does not appear) depends on two factors:</p>

<ul>
 <li>The way the developer writes the plug-in determines whether it is displayed in its own window or is windowless.</li>
 <li>The way the content provider uses HTML elements to invoke the plug-in determines its display mode: whether it is embedded in a page, is part of a section, appears on its own separate page, or is hidden.</li>
</ul>

<p>This section discusses using HTML elements and display modes. For information about windowed and windowless operation, see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Windowed_and_Windowless_Plug-ins">Windowed and Windowless Plug-ins</a>.</p>

<p>For a description of each plug-in display mode, and which HTML element to use to achieve it, go on to <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Plug-in_Display_Modes">Plug-in Display Modes</a>. For details about the HTML elements and their attributes, go on to:</p>

<ul>
 <li><a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Using_the_object_Element_for_Plug-in_Display">Using the object Element for Plug-in Display</a></li>
 <li><a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Using_the_embed_Element_for_Plug-in_Display">Using the embed Element for Plug-in Display</a></li>
</ul>

<h4 id="Plug-in_Display_Modes">Plug-in display modes</h4>

<p>Whether you are writing an HTML page to display a plug-in or developing a plug-in for an HTML author to include in a page, you need to understand how the display mode affects the way plug-ins appear.</p>

<p>A plug-in, whether it is windowed or windowless, can have one of these display modes:</p>

<ul>
 <li>embedded in a web page and visible</li>
 <li>embedded in a web page and hidden</li>
 <li>displayed as a full page in its own window</li>
</ul>

<p>An <strong>embedded plug-in</strong> is part of a larger HTML document and is loaded at the time the document is displayed. The plug-in is visible as a rectangular subpart of the page (unless it is hidden). Embedded plug-ins are commonly used for multimedia images relating to text in the page, such as the Adobe Flash plug-in. When Gecko encounters the <code>object</code> or <code>embed</code> element in a document, it attempts to find and display the file represented by the <code>data</code> and <code>src</code> attributes, respectively. The <code>height</code> and <code>width</code> attributes of the <code>object</code> element determine the size of the embedded plug-in in the HTML page. For example, this <code>object</code> element calls a plug-in that displays video:</p>

<pre class="brush: html">&lt;object data="newave.avi" type="video/avi"
        width="320" height="200"
        autostart="true" loop="true"&gt;
&lt;/object&gt;
</pre>

<p>A <strong>hidden plug-in</strong> is a type of embedded plug-in that is not drawn on the screen when it is invoked. It is created by using the <code>hidden</code> attribute of the <code>embed</code> element. Here's an example:</p>

<pre class="brush: html">&lt;embed src="audiplay.aiff" type="audio/x-aiff" hidden="true"&gt;
</pre>

<div class="note">
<p><strong>Note:</strong> Whether a plug-in is windowed or windowless is not meaningful if the plug-in is invoked with the <code>hidden</code> attribute.</p>
</div>

<p>You can also create hidden plug-ins using the <code>object</code> element. Though the <code>object</code> element has no <code>hidden</code> attribute, you can create <a href="/en-US/docs/Web/CSS">CSS</a> rules to override the sizing attributes of the <code>object</code> element</p>

<pre class="brush: css">object {
  visibility: visible;
}

object.hiddenObject {
  visibility:   hidden !important;
  width:        0px    !important;
  height:       0px    !important;
  margin:       0px    !important;
  padding:      0px    !important;
  border-style: none   !important;
  border-width: 0px    !important;
  max-width:    0px    !important;
  max-height:   0px    !important;
}
</pre>

<p>In this case, the <code>object</code> element that picks up these special style definitions would have a class of hidden. Using the <code>class</code> attribute and the CSS block above, you can simulate the behavior of the hidden plug-in in the <code>embed</code> element:</p>

<pre class="brush: html">&lt;object data="audiplay.aiff" type="audio/x-aiff" class="hiddenObject"&gt;&lt;/object&gt;
</pre>

<p>A <strong>full-page plug-in</strong> is a visible plug-in that is not part of an HTML page. The server looks for the media (MIME) type registered by a plug-in, based on the file extension, and starts sending the file to the browser. Gecko looks up the MIME type and loads the appropriate plug-in if it finds a plug-in registered to that type. This type of plug-in completely fills the web page. Full-page plug-ins are commonly used for document viewers, such as Adobe Acrobat.</p>

<div class="note">
<p><strong>Note:</strong> The browser does not display scroll bars automatically for a full-page plug-in. The plug-in must draw its own scroll bars if it requires them.</p>
</div>

<p>The browser user interface remains relatively constant regardless of which type of plug-in is displayed. The part of the application window that does not display plug-in data does not change. The basic operations of the browser, such as navigation, history, and opening files, apply to all pages, regardless of the plug-ins in use.</p>

<h4 id="Using_the_object_Element_for_Plug-in_Display">Using the <code>object</code> element for plug-in display</h4>

<p>The <a href="/en-US/docs/Web/HTML/Element/object" title="The HTML &lt;object&gt; element represents an external resource, which can be treated as an image, a nested browsing context, or a resource to be handled by a plugin."><code>&lt;object&gt;</code></a> element is part of the HTML specification for generic inclusion of special media in a web page. It embeds a variety of object types in an HTML page, including plug-ins, Java components, ActiveX controls, applets, and images. <code>object</code> element attributes determine the type of object to embed, the type and location of the object's implementation (code), and the type and implementation of the object's data.</p>

<p>Plug-ins were originally designed to work with the <code>embed</code> element rather than the <code>object</code> element (see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Using_the_embed_Element_for_Plug-in_Display">Using the embed Element for Plug-in Display</a>), but the <code>object</code> element itself provides some flexibility here. In particular, the <code>object</code> element allows you to invoke another object if the browser cannot support the object invoked by the element. The <code>embed</code> element, which is also used for plug-ins, does not.</p>

<p>The <code>object</code> element is also a part of the <a class="external" href="https://www.w3.org/TR/html401/struct/objects.html#h-13.3">HTML W3C standard</a>.</p>

<p>Also, unlike the <a href="/en-US/docs/Web/HTML/Element/applet" title="The obsolete HTML Applet Element (&lt;applet&gt;) embeds a Java applet into the document; this element has been deprecated in favor of &lt;object&gt;."><code>&lt;applet&gt;</code></a> element, <code>object</code> can contain other HTML elements including other <code>object</code> elements, nested between its opening and closing tags. So, for example, though Gecko does not support the <code>classid</code> attribute of the <code>object</code> element - which was used for Java classes and ActiveX plug-ins embedded in pages - <code>object</code> elements can be nested to support different plug-in implementations.</p>

<p>See the Mozilla ActiveX project page in the <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Plug-in_References">Plug-in References</a> section below for more information about embedding ActiveX controls in plug-ins or embedding plug-ins in ActiveX applications.</p>

<p>The following examples demonstrate this use of nested <code>object</code> elements with markup more congenial to Gecko included as children of the parent <code>object</code> element.</p>

<p>Example 1: Nesting <code>object</code> Elements</p>

<pre class="brush: html">&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Example 1: Nesting object Elements&lt;/title&gt;
&lt;style type="text/css"&gt;
  .myPlugin {
     width:  470px;
     height: 231px;
  }
&lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;&lt;p&gt;
&lt;object classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
        codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=5,0,30,0"
        class="myPlugin"&gt;

  &lt;param name="movie" value="foo.swf"/&gt;
  &lt;param name="quality" value="high"/&gt;
  &lt;param name="salign" value="tl"/&gt;
  &lt;param name="menu" value="0"/&gt;

       &lt;object data="foo_movie.swf"
               type="application/x-shockwave-flash"
               class="myPlugin"/&gt;

         &lt;param name="quality" value="high"/&gt;
         &lt;param name="salign" value="tl"/&gt;
         &lt;param name="menu" value="0"/&gt;

          &lt;object type="*" class="myPlugin"&gt;
            &lt;param name="pluginspage"
                   value="http://www.macromedia.com/shockwave/download/index.cgi?P1_Prod_Version=ShockwaveFlash"/&gt;
          &lt;/object&gt;

       &lt;/object&gt;
&lt;/object&gt;
&lt;/p&gt;&lt;/body&gt;
&lt;/html&gt;
</pre>

<p>The outermost <code>object</code> element defines the <code>classid</code>; the first nested <code>object</code> uses the <code>type</code> value <code>application/x-shockwave-flash</code> to load the Adobe Flash plug-in, and the innermost <code>object</code> exposes a download page for users that do not already have the necessary plug-in. This nesting is quite common in the use of <code>object</code> elements, and lets you avoid code forking for different browsers.</p>

<h4 id="Nesting_Rules_for_HTML_Elements">Nesting rules for HTML elements</h4>

<p>The rules for descending into nested <code>object</code> and <code>embed</code> elements are as follows:</p>

<ul>
 <li>The browser looks at the MIME type of the top element. If it knows how to deal with that MIME type (i.e., by loading a plug-in that's been registered for it), then it does so.</li>
 <li>If the browser cannot handle the MIME type, it looks in the element for a pointer to a plug-in that can be used to handle that MIME type. The browser downloads the requested plug-in.</li>
 <li>If the MIME type is unknown and there is no reference to a plug-in that can be used, the browser descends into the child element, where these rules for handling MIME types are repeated.</li>
</ul>

<p>The rest of this section is a brief introduction to this HTML element. For more information on the <code>object</code> element and other elements used for plug-in display, see <a class="external" href="https://www.w3.org/TR/html401/struct/objects.html">W3C HTML 4.01 specification</a>.</p>

<p>To embed a variety of object types in an HTML page, use the <code>object</code> element.</p>

<pre class="brush: html">&lt;object
  classid="classFile"
  data="dataLocation"
  codebase="classFileDir"
  type="MIMEtype"
  align="alignment"
  height="pixHeight"
  width="pixWidth"
  id="name"
 &gt;

...

&lt;/object&gt;
</pre>

<p>The first set of <code>object</code> element attributes are URLs.</p>

<ul>
 <li><code>classid</code> is the <code>URL</code> of the specific object implementation. This attribute is similar to the <code>code</code> attribute of the <code>applet</code> element. Though Gecko does not support this <code>object</code> attribute, you can nest <code>object</code> elements with different attributes to use the <code>object</code> element for embedding plug-ins on any browser platform (see the example above).</li>
 <li><code>data</code> represents the <code>URL</code> of the object's data; this is equivalent to the <code>src</code> attribute of <code>embed</code>.</li>
 <li><code>codebase</code> represents the <code>URL</code> of the plug-in; this is the same as the <code>codebase</code> attribute of the <code>applet</code> element. For plug-ins, <code>codebase</code> is the same as <code>pluginspace</code>.</li>
 <li><code>type</code> represents the MIME type of the plug-in; this is the same as the <code>type</code> attribute of <code>embed</code>.</li>
 <li><code>height</code>, <code>width</code>, <code>align</code> are basic <code>img/embed/applet</code> attributes supported by <code>object</code>. <code>height</code> and <code>width</code> are required for <code>object</code> elements that resolve to <code>embed</code> elements.</li>
 <li>Use the <code>id</code> attribute, which specifies the name of the plug-in, if the plug-in is communicating with JavaScript. This is equivalent to the <code>name</code> attribute of <code>applet</code> and <code>embed</code>. It must be unique.</li>
</ul>

<h4 id="Using_the_Appropriate_Attributes">Using the appropriate attributes</h4>

<p>It's up to you to provide enough attributes and to make sure that they do not conflict; for example, the values of <code>width</code> and <code>height</code> may be wrong for the plug-in. Otherwise, the plug-in cannot be embedded.</p>

<p>Gecko interprets the attributes as follows: When the browser encounters an <code>object</code> element, it goes through the element attributes, ignoring or parsing as appropriate. It analyzes the attributes to determine the object type, then determines whether the browser can handle the type.</p>

<ul>
 <li>If the browser can handle the type-that is, if a plug-in exists for that type-then all elements and attributes up to the closing <code>&lt;/object&gt;</code> element, except <code>param</code> elements and other <code>object</code> elements, are filtered.</li>
 <li>If the browser cannot handle the type, or cannot determine the type, it cannot embed the object. Subsequent HTML is parsed as normal.</li>
</ul>

<h4 id="Using_the_embed_Element_for_Plug-in_Display">Using the <code>embed</code> element for plug-in display</h4>

<p>A plug-in runs in an HTML page in a browser window. The HTML author uses the HTML <a href="/en-US/docs/Web/HTML/Element/embed" title="The HTML &lt;embed&gt; element embeds external content at the specified point in the document. This content is provided by an external application or other source of interactive content such as a browser plug-in."><code>&lt;embed&gt;</code></a> element to invoke the plug-in and control its display. Though the <code>object</code> element is the preferred way to invoke plug-ins (see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Basics#Using_the_object_Element_for_Plug-in_Display">Using the object Element for Plug-in Display</a>), the <code>embed</code> element can be used for backward compatibility with Netscape 4.x browsers, and in cases where you specifically want to prompt the user to install a plug-in, because the default plug-in is only automatically invoked when you use the <code>embed</code> element.</p>

<p>Gecko loads an embedded plug-in when the user displays an HTML page that contains an embedded object whose MIME type is registered by a plug-in. Plug-ins are embedded in much the same way as GIF or JPEG images are, except that a plug-in can be live and respond to user events, such as mouse clicks.</p>

<p>The <code>embed</code> element has the following syntax and attributes:</p>

<pre class="brush: html">&lt;embed
  src="location"
  type="mimetype"
  pluginspage="instrUrl"
  pluginurl="pluginUrl"
  align="left"|"right"|"top"|"bottom"
  border="borderWidth"
  frameborder="no"
  height="height"
  width="width"
  units="units"
  hidden="true|false"
  hspace="horizMargin"
  vspace="vertMargin"
  name="pluginName"
  palette="foreground"|"background"
 &gt;

...

&lt;/embed&gt;
</pre>

<p>You must include either the <code>src</code> attribute or the <code>type</code> attribute in an <code>embed</code> element. If you do not, then there is no way of determining the media type, and so no plug-in loads.</p>

<p>The <code>src</code> attribute is the <code>URL</code> of the file to run. The <code>type</code> attribute specifies the MIME type of the plug-in needed to run the file. Navigator uses either the value of the <code>type</code> attribute or the suffix of the filename given as the source to determine which plug-in to use.</p>

<p>Use <code>type</code> to specify the media type or MIME type necessary to display the plug-in. It is good practice to include the MIME type in all the plug-in HTML elements. You can use <code>type</code> for a plug-in that requires no data, for example, a plug-in that draws an analog clock or fetches all of its data dynamically. For a visible plug-in, you must include <code>width</code> and <code>height</code> if you use <code>type</code>; no default value is used.</p>

<p>The <code>pluginurl</code> attribute is the URL of the plug-in or of the XPI in which the plug-in is stored (see <a href="/en-US/docs/Gecko_Plugin_API_Reference/Plug-in_Development_Overview#Installing_Plug-ins">Installing Plug-ins</a> for more information on the XPI file format).</p>

<p>The <code>embed</code> element has a number of attributes that determine the appearance and size of the plug-in instance, including these:</p>

<ul>
 <li>The <code>border</code> and <code>frameborder</code> attributes specify the size of a border for the plug-in or draw a borderless plug-in.</li>
 <li><code>height</code>, <code>width</code>, and <code>units</code> determine the size of the plug-in in the HTML page. If the plug-in is not hidden, the <code>height</code> and <code>width</code> attributes are required.</li>
 <li><code>hspace</code> and <code>vspace</code> create a margin of the specified width, in pixels, around the plug-in.</li>
 <li><code>align</code> specifies the alignment for the plug-in relative to the web page.</li>
</ul>

<p>Use the <code>hidden</code> attribute if you do not want the plug-in to be visible. In this case, you do not need the attributes that describe plug-in appearance. In fact, <code>hidden</code> overrides those attributes if they are present.</p>

<p>Use the <code>name</code> attribute, which specifies the name of the plug-in or plug-in instance, if the plug-in is communicating with JavaScript.</p>

<p>For example, this <code>embed</code> element loads a picture with the imaginary data type dgs:</p>

<pre class="brush: html">&lt;embed src="mypic.dgs" width="320" height="200" border="25" align="right"&gt;
</pre>

<p>Gecko interprets the attributes as follows:</p>

<ul>
 <li><code>src</code>: Load the data file and determine the MIME type of the data.</li>
 <li><code>width</code> and <code>height</code>: Set the area of the page handled by the plug-in to 320 by 200 pixels. In general, use CSS to control the size and location of elements within an HTML page.</li>
 <li><code>border</code>: Draw a border 25 pixels wide around the plug-in.</li>
 <li><code>align</code>: Align the plug-in at the right side of the web page.</li>
</ul>

<p>The following example shows an <code>embed</code> element nested within an <code>object</code> element, which is necessary for browsers that do not support the <code>embed</code> element.</p>

<p>Example 2: <code>embed</code> within <code>object</code></p>

<pre class="brush: html">&lt;object classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
   codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=5,0,30,0"
   width="749" height="68"&gt;

 &lt;param name="movie" value="foo.swf"&gt;
 &lt;param name="quality" value="high"&gt;
 &lt;param name="bgcolor" value="#EEEEEE"&gt;
 &lt;param name="salign" value="tl"&gt;
 &lt;param name="menu" value="0"&gt;

 &lt;embed src="foo.swf"
   quality="high" pluginspage="http://www.macromedia.com/shockwave/download/index.cgi?P1_Prod_Version=ShockwaveFlash"
   type="application/x-shockwave-flash"
   width="749"
   height="68"
   bgcolor="#EEEEEE"
   salign="tl"
   menu="0"&gt;

 &lt;/embed&gt;

&lt;/object&gt;
</pre>

<h4 id="Using_Custom_embed_Attributes">Using custom <code>embed</code> attributes</h4>

<p>In addition to these standard attributes, you can create private, plug-in-specific attributes and use them in the <code>embed</code> attribute to pass extra information between the HTML page and the plug-in code. The browser ignores these nonstandard attributes when parsing the HTML, but it passes all attributes to the plug-in, allowing the plug-in to examine the list for any private attributes that could modify its behavior.</p>

<p>For example, a plug-in that displays video could have private attributes that determine whether to start the plug-in automatically or loop the video automatically on playback, as in the following <code>embed</code> element:</p>

<pre class="brush: html">&lt;embed src="myavi.avi" width="100" height="125" autostart="true" loop="true"&gt;
</pre>

<p><br>
 With this <code>embed</code> element, Gecko passes the values to the plug-in, using the arg parameters of the <code>NPP_New</code> call that creates the plug-in instance.</p>

<pre class="eval">argc = 5
argn = {"src", "width", "height", "autostart", "loop"}
argv = {"movie.avi", "100", "125", "true", "true"}
</pre>

<p>Gecko interprets the attributes as follows:</p>

<ul>
 <li><code>src</code>: Load the data file and determine the MIME type of the data.</li>
 <li><code>width</code> and <code>height</code>: Set the area of the page handled by the plug-in to 100 by 125 pixels.</li>
 <li><code>autostart</code> and <code>loop</code>: Ignore these private attributes and pass them along to the plug-in with the rest of the attributes.</li>
</ul>

<p>The plug-in must scan its list of attributes to determine whether it should automatically start the video and loop it on playback. Note that with an <code>object</code> element, <code>param</code> values are also sent in this array after the attributes, separated by a <code>param</code> entry.</p>

<h3 id="Plug-in_References">Plug-in references</h3>

<ul>
 <li><a class="external" href="https://www.mozilla.org/projects/plugins/">The Mozilla Plug-ins project page</a></li>
</ul>