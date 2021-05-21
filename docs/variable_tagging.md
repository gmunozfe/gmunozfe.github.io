## jBPM Variable tagging

ifdef::env-github[]
:tip-caption: :bulb:
:note-caption: :information_source:
:important-caption: :heavy_exclamation_mark:
:caution-caption: :fire:
:warning-caption: :warning:
endif::[]

Variable tagging allows developers to hint jBPM for extra behavior to a specific process variable. Tags can be used by process or case instances. Tags are simple String values that are added as metadata to a specific variable. A process/case instance variable can have multiple tags associated with it.
By default, jBPM supports a few out-of-the-box tags.

IMPORTANT: You're not limited to these out-of-the-box tags. You can also create your own custom tags.

## Out-of-the-box tags
These tags are predefined and can be selected directly.

### Possible values:
* **required** - A process variable tagged as _required_, instructs the jBPM engine that once starting a new instance, this variable must be supplied. Same for case management, the _required_ variable must be present when starting a case. Otherwise, a _VariableViolationException_ is thrown.
* **readonly** - A process variable tagged as _readonly_, can only be set once during the process instance execution. In the same way, for case management, _readonly_ variables can only be assigned once (either before starting the case or after started). If at any time, there is an attempt to change its value, a _VariableViolationException_ is thrown.

## Custom tags
As a developer, you can add any String as a tag for a process/case instance variable, apart from out-of-the-box ones. It allows that this information is then available via event listeners. This is due to `ProcessVariableChangedEvent` interface has added these two methods:

[source, java]
----
/**
 * List of tags associated with variable that is being changed.
 * @return list of tags if there are any otherwise empty list
 */
List<String> getTags();

/**
 * Determines if variable that is being changed has given tag associated with it
 * @param tag name of the tag
 * @return returns true if given tag is associated with variable otherwise false
 */
boolean hasTag(String tag);
----

Custom tags may be also added to the process or case by means of Business Central, introducing a non-predefined value for the target variable. 

image::https://user-images.githubusercontent.com/1962786/78351028-acecd300-7595-11ea-8729-7e2ab1e575b5.png[]


TIP: Variable tags can also be handled into the _bpmn_ files, as they are defined as _metaData_ under the _customTags_ node

[source, xml]
----
<bpmn2:property id="myTaggedVariable" itemSubjectRef="_myTaggedVariableItem">
  <bpmn2:extensionElements>
    <drools:metaData name="customTags">
      <drools:metaValue><![CDATA[required, myTag]]></drools:metaValue> <!--1-->
    </drools:metaData>
  </bpmn2:extensionElements>
</bpmn2:property>   
----
<1> _multiple tags (out-of-the-box and/or custom-specific) are allowed_

## Security restriction tags
Other useful behavior for variable tagging is to enforce any security restrictions based on custom tags associated with these variables. Again, this can be achieved by registering an additional event listener that can react on _beforeVariableChange_ event to apply different security constraints.

There is a special tag, named _**restricted**_, that can be used with the provided `VariableGuardProcessEventListener` for granted permission to modify that variable based on the required role and the current role of the user (obtained from the configured Identity Provider).

This `VariableGuardProcessEventListener` extends from `DefaultProcessEventListener` and supports two different constructors (with explicit custom tag or without it -i.e, considers _restricted_ as default tag): 

[source, java]
----
public VariableGuardProcessEventListener(String requiredRole, IdentityProvider identityProvider) {        
    this("restricted", requiredRole, identityProvider);
}

public VariableGuardProcessEventListener(String tag, String requiredRole, IdentityProvider identityProvider) {
    this.tag = tag;
    this.requiredRole = requiredRole;
    this.identityProvider = identityProvider;
}
----

So, basically the event listener must be added to the session with the allowed role name and the identity provider that returns the user role: 

[source, java]
----
ksession.addEventListener(new VariableGuardProcessEventListener("AdminRole", myIdentityProvider));
----

`VariableGuardProcessEventListener` will react before variable change to check if the variable has tagged with a security constraint tag and in this case, it will raise a _VariableViolationException_ if the user role is not the required one.


NOTE: For case management, security policies for variables implemented through `VariableGuardProcessEventListener` are executed before starting a case, reopening it or adding data to a case file. 

CAUTION: For other custom tags, it is possible also to react after variable change in the same scenarios by implementing `afterVariableChanged` in the custom event listener.

TIP: https://github.com/kiegroup/kogito-runtimes[Kogito] (next generation of cloud-native business automation for building intelligent applications as evolution of Drools, jBPM and OptaPlanner) supports more variable tagging out-of-the-box: `internal`, `input`, `output`, `business-relevant` and `tracked`.


You can use the [editor on GitHub](https://github.com/gmunozfe/gmunozfe.github.io/edit/main/docs/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/gmunozfe/gmunozfe.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
