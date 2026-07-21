## Main issue in your migrated code

You changed:

```xhtml
<a:commandButton>
```

to:

```xhtml
<p:commandButton>
```

but you kept:

```xhtml
image="/resources/images/common/button-previous.png"
```

That is the primary problem.

`p:commandButton` in PrimeFaces 15 does **not** have an `image` attribute. It supports `icon`, `iconPos`, `process`, `update`, `disabled`, `rendered`, and `disableOnAjax`, but not RichFaces-style `image`. ([PrimeFaces][1])

Therefore, this code is invalid:

```xhtml
<p:commandButton
    image="/resources/images/common/button-previous.png">
</p:commandButton>
```

Depending on your configuration, you may see:

```text
Attribute image is not defined for component commandButton
```

or the page may display an empty PrimeFaces button.

---

# Other issues visible in your code

## 1. Check the spelling of `disablePrev`

Your screenshot appears to contain:

```xhtml
.diablePrev
```

The `s` may be missing.

You currently seem to have:

```xhtml
#{independentContactHistoryController
    .viewEditFollowUpListBean.diablePrev}
```

Confirm the Java getter.

When Java has:

```java
public boolean isDisablePrev() {
    return disablePrev;
}
```

XHTML must use:

```xhtml
#{independentContactHistoryController
    .viewEditFollowUpListBean.disablePrev}
```

Not:

```xhtml
.diablePrev
```

However, if your Java property itself is intentionally named `diablePrev`, the current expression will work, although the property should preferably be corrected.

---

## 2. Remove manual JavaScript disabling

You currently have:

```xhtml
onclick="this.disabled=true;"
```

Remove it.

PrimeFaces `p:commandButton` already has:

```xhtml
disableOnAjax="true"
```

and its default value is already `true`. PrimeFaces disables the button during its Ajax request and enables it again when the request completes. ([PrimeFaces][1])

Manual disabling can create problems:

```text
User clicks button
    ↓
Browser disables button immediately
    ↓
Disabled control may not be included correctly in submission
    ↓
Action method may not execute
```

Use:

```xhtml
disableOnAjax="true"
```

instead.

---

## 3. Boolean comparison is incorrect or fragile

You currently have something similar to:

```xhtml
#{independentContactHistoryController.dataModified == 'false'}
```

Here `'false'` is a **String**.

When `dataModified` is a Java boolean, use:

```xhtml
#{not independentContactHistoryController.dataModified}
```

or:

```xhtml
#{independentContactHistoryController.dataModified eq false}
```

However, using EL directly inside `oncomplete` can give you an old value because the JavaScript was generated before the current Ajax action completed.

A PrimeFaces callback parameter is more reliable, as shown later.

---

## 4. Verify your update IDs

You appear to have:

```xhtml
update=":viewEditContactHistoryForm:editFollowupNoteMainID
        :editCloseFollowUpConfirmDialog
        ..."
```

This is correct only when:

```text
editCloseFollowUpConfirmDialog
```

exists directly under the JSF view root.

When the dialog is inside:

```xhtml
<h:form id="viewEditContactHistoryForm">
```

the complete absolute ID should be:

```xhtml
:viewEditContactHistoryForm:editCloseFollowUpConfirmDialog
```

For example:

```xhtml
update=":viewEditContactHistoryForm:editFollowupNoteMainID
        :viewEditContactHistoryForm:editCloseFollowUpConfirmDialog
        :viewEditContactHistoryForm:contactViewEditSaveCloseButtonsPanelID"
```

When all components are in the same form and naming container, relative IDs are simpler:

```xhtml
update="editFollowupNoteMainID
        editCloseFollowUpConfirmDialog
        contactViewEditSaveCloseButtonsPanelID"
```

---

# Four practical ways to migrate this image button

## Option 1: `p:commandLink` with `h:graphicImage`

This is the **best option for your application** because you want to retain the existing previous-arrow PNG images.

Use an outer panel that is always rendered. Inside it:

* Show a plain disabled image when previous navigation is unavailable.
* Show a clickable `p:commandLink` with an image when navigation is available.

```xhtml
<h:panelGroup id="previousButtonContainer"
              layout="block">

    <!-- Disabled previous arrow -->
    <h:graphicImage
        library="images"
        name="common/button-previous-disabled.png"
        alt="Previous record unavailable"
        title="Previous record unavailable"
        styleClass="history-arrow-disabled"
        rendered="#{independentContactHistoryController
                    .viewEditFollowUpListBean.disablePrev}"/>

    <!-- Enabled previous arrow -->
    <p:commandLink
        id="prvImg"
        rendered="#{not independentContactHistoryController
                     .viewEditFollowUpListBean.disablePrev}"

        action="#{independentContactHistoryController
                   .getContactHistoryForNoteIDPreNext('prev')}"

        process="@form"

        update="editFollowupNoteMainID
                editCloseFollowUpConfirmDialog
                contactViewEditSaveCloseButtonsPanelID
                previousButtonContainer"

        disableOnAjax="true"

        ariaLabel="Previous contact history"

        oncomplete="handlePreviousNavigation(args)">

        <h:graphicImage
            library="images"
            name="common/button-previous.png"
            alt="Previous contact history"
            title="Previous contact history"
            styleClass="history-arrow"/>

    </p:commandLink>

</h:panelGroup>
```

Jakarta Faces `h:graphicImage` supports `library` and `name`; Faces resolves the resource and generates the correct application-aware URL. ([Jakarta EE][2])

### Folder structure

```text
src/main/webapp/resources/
└── images/
    └── common/
        ├── button-previous.png
        └── button-previous-disabled.png
```

### Important point

Update this parent:

```xhtml
previousButtonContainer
```

Do not update only:

```xhtml
prvImg
```

When `rendered="false"`, the component does not exist in the browser HTML. Updating the permanently rendered parent allows JSF to replace the disabled image with the enabled button, or vice versa.

### JavaScript

```xhtml
<script type="text/javascript">
    function handlePreviousNavigation(args) {
        setDatepicker();

        if (args &amp;&amp; args.dataModified) {
            showConfirmationContactHistoryPopup(
                'viewEditPrevFollowUpConfirmDialog',
                400
            );
        }
    }
</script>
```

Because this is XHTML, use:

```text
&amp;&amp;
```

not:

```text
&&
```

inside XML content or component attributes.

### Java callback parameter

Inside your action logic:

```java
import org.primefaces.PrimeFaces;

public void getContactHistoryForNoteIDPreNext(String direction) {

    // Existing previous/next navigation logic

    PrimeFaces.current()
            .ajax()
            .addCallbackParam("dataModified", dataModified);
}
```

Then this receives the latest server-side value:

```javascript
args.dataModified
```

---

# Option 2: One `p:commandLink` with a dynamic image

Instead of keeping two components, you can use one component and change its image dynamically.

```xhtml
<h:panelGroup id="previousButtonContainer">

    <p:commandLink
        id="prvImg"

        disabled="#{independentContactHistoryController
                    .viewEditFollowUpListBean.disablePrev}"

        action="#{independentContactHistoryController
                   .getContactHistoryForNoteIDPreNext('prev')}"

        process="@form"

        update="editFollowupNoteMainID
                editCloseFollowUpConfirmDialog
                contactViewEditSaveCloseButtonsPanelID
                previousButtonContainer"

        disableOnAjax="true"

        ariaLabel="Previous contact history"

        oncomplete="handlePreviousNavigation(args)">

        <h:graphicImage
            library="images"

            name="#{independentContactHistoryController
                    .viewEditFollowUpListBean.disablePrev
                    ? 'common/button-previous-disabled.png'
                    : 'common/button-previous.png'}"

            alt="Previous contact history"
            styleClass="history-arrow"/>

    </p:commandLink>

</h:panelGroup>
```

### Benefits

```text
Only one command component
No duplicate action/update/process definitions
Image changes based on disablePrev
Easier maintenance
```

This is also a good solution.

My preference between the first two:

* Choose **Option 1** when the disabled arrow should be a plain non-clickable image.
* Choose **Option 2** when you prefer fewer components.

---

# Option 3: Use a PrimeFaces icon

When you do not need the exact old PNG, replace it with a PrimeIcon:

```xhtml
<p:commandButton
    id="prvImg"

    icon="pi pi-chevron-left"

    title="Previous contact history"
    ariaLabel="Previous contact history"

    styleClass="ui-button-flat ui-button-secondary"

    disabled="#{independentContactHistoryController
                .viewEditFollowUpListBean.disablePrev}"

    action="#{independentContactHistoryController
               .getContactHistoryForNoteIDPreNext('prev')}"

    process="@form"

    update="editFollowupNoteMainID
            editCloseFollowUpConfirmDialog
            contactViewEditSaveCloseButtonsPanelID
            previousButtonContainer"

    disableOnAjax="true"

    oncomplete="handlePreviousNavigation(args)"/>
```

PrimeFaces 15 officially supports the `icon` attribute on `p:commandButton`. ([PrimeFaces][1])

Possible icons:

```xhtml
icon="pi pi-chevron-left"
icon="pi pi-angle-left"
icon="pi pi-arrow-left"
icon="pi pi-step-backward"
```

In this option you no longer need:

```text
button-previous.png
button-previous-disabled.png
```

PrimeFaces automatically applies disabled styling.

---

# Option 4: Standard Jakarta Faces `h:commandButton`

Jakarta Faces itself supports an image-style command button.

```xhtml
<h:commandButton
    id="prvImg"

    image="#{resource['images:common/button-previous.png']}"

    disabled="#{independentContactHistoryController
                .viewEditFollowUpListBean.disablePrev}"

    action="#{independentContactHistoryController
               .getContactHistoryForNoteIDPreNext('prev')}">

    <f:ajax
        execute="@form"
        render="editFollowupNoteMainID
                editCloseFollowUpConfirmDialog
                contactViewEditSaveCloseButtonsPanelID
                previousButtonContainer"
        onevent="handleFacesPreviousAjax"/>

</h:commandButton>
```

JavaScript:

```javascript
function handleFacesPreviousAjax(data) {
    if (data.status === 'success') {
        setDatepicker();
    }
}
```

This is the closest standard-JSF replacement for your old image command button, but you lose some convenient PrimeFaces features such as `process`, `update`, `oncomplete`, and PrimeFaces callback arguments. Standard Faces Ajax instead uses `execute`, `render`, and `onevent`. ([Jakarta EE][3])

For your popup workflow, a PrimeFaces command component is more convenient.

---

# Can you keep two `p:commandButton` components?

You can keep two components, but you still cannot use `image`.

You could use custom CSS icons:

```xhtml
<h:panelGroup id="previousButtonContainer">

    <p:commandButton
        id="prvImgDisabled"
        icon="custom-previous-disabled-icon"
        disabled="true"
        rendered="#{independentContactHistoryController
                    .viewEditFollowUpListBean.disablePrev}"
        styleClass="secondary image-only-button"
        ariaLabel="Previous record unavailable"/>

    <p:commandButton
        id="prvImg"
        icon="custom-previous-icon"

        rendered="#{not independentContactHistoryController
                     .viewEditFollowUpListBean.disablePrev}"

        action="#{independentContactHistoryController
                   .getContactHistoryForNoteIDPreNext('prev')}"

        process="@form"

        update="editFollowupNoteMainID
                editCloseFollowUpConfirmDialog
                contactViewEditSaveCloseButtonsPanelID
                previousButtonContainer"

        disableOnAjax="true"

        ariaLabel="Previous contact history"

        oncomplete="handlePreviousNavigation(args)"/>

</h:panelGroup>
```

CSS:

```css
.custom-previous-icon,
.custom-previous-disabled-icon {
    width: 20px;
    height: 20px;
    display: inline-block;
    background-repeat: no-repeat;
    background-position: center;
    background-size: contain;
}

.custom-previous-icon {
    background-image:
        url("../images/common/button-previous.png");
}

.custom-previous-disabled-icon {
    background-image:
        url("../images/common/button-previous-disabled.png");
}

.image-only-button {
    padding: 0;
    border: none;
    background: transparent;
}
```

This works, but `p:commandLink` with `h:graphicImage` is simpler for existing PNG files.

---

# `process="@form"` may prevent your action method

Your migration from:

```xhtml
execute="@form"
```

to:

```xhtml
process="@form"
```

is technically the correct attribute mapping.

However, `process="@form"` processes and validates every form field.

Suppose one input is:

```xhtml
<p:inputText required="true"/>
```

and it is empty.

Then:

```text
User clicks previous arrow
    ↓
Entire form is processed
    ↓
Required validation fails
    ↓
Invoke Application is skipped
    ↓
getContactHistoryForNoteIDPreNext("prev") is not called
```

Choose the processing scope based on your requirement.

### Process only the arrow

```xhtml
process="@this"
```

Use this when previous/next navigation does not need form values.

### Process the editable content

```xhtml
process="editFollowupNoteMainID"
```

Use this when you need the currently edited note fields.

### Process the entire form

```xhtml
process="@form"
```

Use this only when all form inputs must be converted and validated.

---

# Recommended final code for your application

I recommend this version:

```xhtml
<h:panelGroup id="previousButtonContainer"
              layout="block">

    <h:graphicImage
        library="images"
        name="common/button-previous-disabled.png"
        alt="Previous record unavailable"
        rendered="#{independentContactHistoryController
                    .viewEditFollowUpListBean.disablePrev}"/>

    <p:commandLink
        id="prvImg"

        rendered="#{not independentContactHistoryController
                     .viewEditFollowUpListBean.disablePrev}"

        action="#{independentContactHistoryController
                   .getContactHistoryForNoteIDPreNext('prev')}"

        process="@form"

        update="editFollowupNoteMainID
                editCloseFollowUpConfirmDialog
                contactViewEditSaveCloseButtonsPanelID
                previousButtonContainer"

        disableOnAjax="true"

        ariaLabel="Previous contact history"

        oncomplete="handlePreviousNavigation(args)">

        <h:graphicImage
            library="images"
            name="common/button-previous.png"
            alt="Previous contact history"/>

    </p:commandLink>

</h:panelGroup>
```

## Migration summary

| Current code                          | Problem or change                                       |
| ------------------------------------- | ------------------------------------------------------- |
| `p:commandButton image="..."`         | Invalid; `image` is not supported                       |
| `execute="@form"`                     | Change to `process="@form"`                             |
| `render="..."`                        | Change to `update="..."`                                |
| `onclick="this.disabled=true"`        | Remove it                                               |
| `disableOnAjax="true"`                | Use PrimeFaces built-in disabling                       |
| `.diablePrev`                         | Check whether this is a spelling mistake                |
| `dataModified == 'false'`             | Do not compare Boolean with String                      |
| Two image command buttons             | Replace with image + command link, or one dynamic image |
| Updating a `rendered=false` component | Update its always-rendered parent panel                 |

[1]: https://primefaces.github.io/primefaces/vdldoc/primefaces/commandButton.html?utm_source=chatgpt.com "commandButton (PrimeFaces VDL Documentation)"
[2]: https://jakarta.ee/specifications/faces/4.0/vdldoc/h/graphicimage?utm_source=chatgpt.com "graphicImage (Jakarta Faces 4.0.0 VDL Documentation)"
[3]: https://jakarta.ee/learn/docs/jakartaee-tutorial/current/web/faces-ajax/faces-ajax.html?utm_source=chatgpt.com "Using Ajax with Jakarta Faces Technology"
