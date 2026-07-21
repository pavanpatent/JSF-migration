We will use **one `p:commandLink` containing one `h:graphicImage`**, with the image selected dynamically based on your existing `.diablePrev` property.

This is the best migration for your case because:

* It preserves your existing PNG arrow images.
* It removes duplicate enabled/disabled command components.
* It keeps PrimeFaces Ajax features such as `process`, `update`, `oncomplete`, and `disableOnAjax`.
* It avoids the unsupported `image` attribute on `p:commandButton`.

PrimeFaces `p:commandButton` supports `icon`, but not an `image` attribute. PrimeFaces `p:commandLink` supports `action`, `disabled`, `process`, `update`, `oncomplete`, and `disableOnAjax`, and it can contain an `h:graphicImage`. ([primefaces.github.io][1])

I will use your actual property name:

```xhtml
.diablePrev
```

# Final design

Instead of this:

```text
Disabled p:commandButton
+
Enabled p:commandButton
```

we will have:

```text
One p:commandLink
    └── One h:graphicImage
            ├── disabled image when diablePrev = true
            └── normal image when diablePrev = false
```

---

# Step 1: Verify the JSF 4 namespaces

At the top of your XHTML page, use:

```xhtml
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="jakarta.faces.html"
      xmlns:f="jakarta.faces.core"
      xmlns:ui="jakarta.faces.facelets"
      xmlns:p="http://primefaces.org/ui">
```

For Jakarta Faces 4, the Faces namespaces use `jakarta.faces.*`. The PrimeFaces namespace remains `http://primefaces.org/ui`. ([primefaces.github.io][2])

Do not use the old JSF namespaces:

```xhtml
xmlns:h="http://java.sun.com/jsf/html"
xmlns:f="http://java.sun.com/jsf/core"
xmlns:ui="http://java.sun.com/jsf/facelets"
```

---

# Step 2: Keep the existing images in the resource folder

Your images should remain here:

```text
src/main/webapp/resources/images/common/
```

The complete structure should be:

```text
src
└── main
    └── webapp
        └── resources
            └── images
                └── common
                    ├── button-previous.png
                    └── button-previous-disabled.png
```

Do not directly write:

```xhtml
image="/resources/images/common/button-previous.png"
```

Instead, use the Jakarta Faces resource mechanism:

```xhtml
<h:graphicImage
    library="images"
    name="common/button-previous.png"/>
```

`h:graphicImage` uses `library` and `name` to resolve the resource and generate the correct URL for the application. ([Jakarta EE][3])

---

# Step 3: Delete both existing `p:commandButton` blocks

Delete this disabled button:

```xhtml
<p:commandButton
    styleClass="secondary"
    id="prvImgDisabled"
    image="/resources/images/common/button-previous-disabled.png"
    disabled="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diablePrev}"
    rendered="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diablePrev}"
    onclick="this.disabled=true;"
    action="#{independentContactHistoryController
               .getContactHistoryForNoteIDPreNext('prev')}"
    process="@form"
    update="..."
    oncomplete="...">
</p:commandButton>
```

Also delete the enabled button:

```xhtml
<p:commandButton
    styleClass="secondary"
    id="prvImg"
    image="/resources/images/common/button-previous.png"
    disabled="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diablePrev}"
    rendered="#{!independentContactHistoryController
                  .viewEditFollowUpListBean.diablePrev}"
    onclick="this.disabled=true;"
    action="#{independentContactHistoryController
               .getContactHistoryForNoteIDPreNext('prev')}"
    process="@form"
    update="..."
    oncomplete="...">
</p:commandButton>
```

Both components will be replaced by one component.

---

# Step 4: Add one always-rendered parent panel

Inside your existing:

```xhtml
<h:panelGroup>
```

add another panel with an ID:

```xhtml
<h:panelGroup id="previousNavigationPanel"
              layout="block">

</h:panelGroup>
```

This parent panel gives us a stable component to update after Ajax.

Why this matters:

```text
Click Previous
    ↓
Server changes diablePrev
    ↓
PrimeFaces updates previousNavigationPanel
    ↓
The correct normal or disabled image is rendered
```

The panel itself always exists, so PrimeFaces can reliably update it.

---

# Step 5: Add the single `p:commandLink`

Place this code inside `previousNavigationPanel`:

```xhtml
<p:commandLink
    id="prvImg"

    styleClass="history-navigation-link"

    title="Previous contact history"

    ariaLabel="Previous contact history"

    disabled="#{independentContactHistoryController
                .viewEditFollowUpListBean.diablePrev}"

    action="#{independentContactHistoryController
              .navigatePreviousContactHistory}"

    process="@form"

    update=":viewEditContactHistoryForm:editFollowupNoteMainID
            :viewEditContactHistoryForm:contactViewEditSaveCloseButtonsPanelID
            :viewEditContactHistoryForm:previousNavigationPanel
            :editCloseFollowUpConfirmDialog"

    disableOnAjax="true"

    oncomplete="handlePreviousNavigationComplete(args)">

</p:commandLink>
```

Do not close it immediately yet. The image goes inside it.

---

# Step 6: Add the dynamic image inside `p:commandLink`

Inside the `p:commandLink`, add:

```xhtml
<h:graphicImage
    library="images"

    name="#{independentContactHistoryController
            .viewEditFollowUpListBean.diablePrev
            ? 'common/button-previous-disabled.png'
            : 'common/button-previous.png'}"

    alt="Previous contact history"

    styleClass="history-navigation-image"/>
```

The completed command link becomes:

```xhtml
<p:commandLink
    id="prvImg"

    styleClass="history-navigation-link"

    title="Previous contact history"

    ariaLabel="Previous contact history"

    disabled="#{independentContactHistoryController
                .viewEditFollowUpListBean.diablePrev}"

    action="#{independentContactHistoryController
              .navigatePreviousContactHistory}"

    process="@form"

    update=":viewEditContactHistoryForm:editFollowupNoteMainID
            :viewEditContactHistoryForm:contactViewEditSaveCloseButtonsPanelID
            :viewEditContactHistoryForm:previousNavigationPanel
            :editCloseFollowUpConfirmDialog"

    disableOnAjax="true"

    oncomplete="handlePreviousNavigationComplete(args)">

    <h:graphicImage
        library="images"

        name="#{independentContactHistoryController
                .viewEditFollowUpListBean.diablePrev
                ? 'common/button-previous-disabled.png'
                : 'common/button-previous.png'}"

        alt="Previous contact history"

        styleClass="history-navigation-image"/>

</p:commandLink>
```

---

# Step 7: Understand the dynamic image condition

This expression:

```xhtml
#{bean.diablePrev
    ? 'common/button-previous-disabled.png'
    : 'common/button-previous.png'}
```

means:

```text
If diablePrev = true
    use button-previous-disabled.png

Otherwise
    use button-previous.png
```

Therefore, you no longer need:

```xhtml
rendered="#{bean.diablePrev}"
```

or:

```xhtml
rendered="#{!bean.diablePrev}"
```

The command link is always rendered.

Only these two things change:

```text
disabled attribute
image filename
```

---

# Step 8: Remove `onclick="this.disabled=true;"`

Do not copy this old line:

```xhtml
onclick="this.disabled=true;"
```

Remove it completely.

Use:

```xhtml
disableOnAjax="true"
```

PrimeFaces disables the link while its Ajax request is running. The documented default is already `true`, but keeping it explicitly makes the migration intention clear. ([primefaces.github.io][1])

The old manual JavaScript could cause the element to remain disabled if:

* Validation fails.
* The Ajax request fails.
* A JavaScript error occurs.
* The server returns an unexpected response.

So this:

```xhtml
onclick="this.disabled=true;"
```

becomes:

```xhtml
disableOnAjax="true"
```

---

# Step 9: Keep `process="@form"` initially

Your old RichFaces code had:

```xhtml
execute="@form"
```

The equivalent PrimeFaces attribute is:

```xhtml
process="@form"
```

Keep it initially so the migration preserves the existing behavior.

```text
RichFaces execute="@form"
            ↓
PrimeFaces process="@form"
```

Do not change it to `process="@this"` during the initial migration unless you confirm that the action does not require any form values.

`process` identifies the components that participate in the partial request processing. ([primefaces.github.io][1])

## Important validation note

Because you process the complete form:

```xhtml
process="@form"
```

all input validations can run.

For example:

```xhtml
<p:inputText required="true"/>
```

If that field is empty:

```text
Click Previous
    ↓
The complete form is processed
    ↓
Required validation fails
    ↓
Action method may not execute
```

For now, preserve `@form`. Test the behavior before optimizing it.

---

# Step 10: Update the arrow panel itself

You must include:

```xhtml
:viewEditContactHistoryForm:previousNavigationPanel
```

in the `update` attribute.

Without it:

```text
Server changes diablePrev = true
    ↓
Main form data changes
    ↓
Arrow is not re-rendered
    ↓
Browser may continue showing the enabled arrow
```

With it:

```text
Server changes diablePrev = true
    ↓
previousNavigationPanel is re-rendered
    ↓
disabled image appears
    ↓
commandLink becomes disabled
```

---

# Step 11: Check the confirmation-dialog client ID

Your current update contains:

```xhtml
:editCloseFollowUpConfirmDialog
```

Use that only when the dialog is outside the form and its client ID is actually:

```text
editCloseFollowUpConfirmDialog
```

For example:

```xhtml
<h:form id="viewEditContactHistoryForm">
    ...
</h:form>

<p:confirmDialog id="editCloseFollowUpConfirmDialog">
    ...
</p:confirmDialog>
```

Then this is correct:

```xhtml
:editCloseFollowUpConfirmDialog
```

However, when the dialog is inside the form:

```xhtml
<h:form id="viewEditContactHistoryForm">

    <p:confirmDialog id="editCloseFollowUpConfirmDialog">
        ...
    </p:confirmDialog>

</h:form>
```

the update ID must be:

```xhtml
:viewEditContactHistoryForm:editCloseFollowUpConfirmDialog
```

Therefore, verify where the confirmation dialog is placed.

---

# Step 12: Do not evaluate `dataModified` directly inside `oncomplete`

Your current code appears similar to:

```xhtml
oncomplete="
    setDatepicker();

    if (#{independentContactHistoryController.dataModified == 'false'}) {
    } else {
        showConfirmationContactHistoryPopup(
            'viewEditPrevFollowUpConfirmDialog',
            400
        );
    }
"
```

There are two problems.

## Problem 1: Boolean compared with String

This:

```xhtml
dataModified == 'false'
```

compares against the String:

```text
"false"
```

It is not the Java boolean:

```java
false
```

## Problem 2: The value can be generated before the Ajax action

The EL expression in the JavaScript is rendered when JSF generates the page or updated component. It may not reliably represent the value generated during the current action.

The better approach is to send a PrimeFaces callback parameter from Java. PrimeFaces makes server-side callback parameters available through the JavaScript Ajax `args` object. ([primefaces.github.io][4])

---

# Step 13: Add a wrapper method in the Java controller

Do not change your existing business method:

```java
getContactHistoryForNoteIDPreNext("prev");
```

Add a small UI wrapper method:

```java
import org.primefaces.PrimeFaces;

public void navigatePreviousContactHistory() {

    getContactHistoryForNoteIDPreNext("prev");

    PrimeFaces.current()
            .ajax()
            .addCallbackParam(
                    "dataModified",
                    isDataModified()
            );
}
```

If your existing getter is:

```java
public boolean getDataModified()
```

then use:

```java
getDataModified()
```

instead:

```java
public void navigatePreviousContactHistory() {

    getContactHistoryForNoteIDPreNext("prev");

    PrimeFaces.current()
            .ajax()
            .addCallbackParam(
                    "dataModified",
                    getDataModified()
            );
}
```

Do not create both versions. Use whichever getter already exists in your controller.

## Why use a wrapper method?

Before:

```xhtml
action="#{independentContactHistoryController
          .getContactHistoryForNoteIDPreNext('prev')}"
```

After:

```xhtml
action="#{independentContactHistoryController
          .navigatePreviousContactHistory}"
```

The wrapper:

1. Calls your existing navigation logic.
2. Reads the latest `dataModified` value.
3. Sends that value to JavaScript.
4. Does not force you to rewrite the existing business method.

---

# Step 14: Create the JavaScript completion function

Add this once near the bottom of the XHTML page:

```xhtml
<h:outputScript target="body">

    function handlePreviousNavigationComplete(args) {

        setDatepicker();

        if (args) {

            if (args.dataModified === true) {

                showConfirmationContactHistoryPopup(
                    'viewEditPrevFollowUpConfirmDialog',
                    400
                );
            }
        }
    }

</h:outputScript>
```

Now the command link uses:

```xhtml
oncomplete="handlePreviousNavigationComplete(args)"
```

The flow becomes:

```text
User clicks Previous
    ↓
navigatePreviousContactHistory() executes
    ↓
Existing getContactHistoryForNoteIDPreNext("prev") executes
    ↓
Java sends dataModified through callback parameter
    ↓
PrimeFaces invokes oncomplete
    ↓
handlePreviousNavigationComplete(args)
    ↓
If args.dataModified is true
    ↓
Confirmation popup opens
```

---

# Step 15: Add CSS for the image link

Create or update:

```text
src/main/webapp/resources/css/contact-history.css
```

Add:

```css
.history-navigation-link {
    display: inline-flex;
    align-items: center;
    justify-content: center;

    padding: 0;
    margin: 0;

    border: 0;
    background: transparent;
    text-decoration: none;

    line-height: 0;
    cursor: pointer;
}

.history-navigation-image {
    display: block;
    border: 0;
}

.history-navigation-link.ui-state-disabled {
    opacity: 1;
    cursor: default;
}
```

The line:

```css
opacity: 1;
```

is useful because you already have a special disabled image:

```text
button-previous-disabled.png
```

Otherwise, PrimeFaces CSS may further fade the already-disabled-looking image.

Load the stylesheet:

```xhtml
<h:outputStylesheet
    library="css"
    name="contact-history.css"/>
```

---

# Complete final XHTML code

This is the complete recommended replacement for the previous-arrow section:

```xhtml
<ui:define name="main-content">

    <div id="render_ContactHistoryViewEditNote"
         title="Edit Independent Client Contact">

        <h:form id="viewEditContactHistoryForm">

            <!-- Existing form content -->

            <div class="data data-title">

                <h:panelGroup id="previousNavigationPanel"
                              layout="block">

                    <p:commandLink
                        id="prvImg"

                        styleClass="history-navigation-link"

                        title="Previous contact history"

                        ariaLabel="Previous contact history"

                        disabled="#{independentContactHistoryController
                                    .viewEditFollowUpListBean.diablePrev}"

                        action="#{independentContactHistoryController
                                  .navigatePreviousContactHistory}"

                        process="@form"

                        update=":viewEditContactHistoryForm:editFollowupNoteMainID
                                :viewEditContactHistoryForm:contactViewEditSaveCloseButtonsPanelID
                                :viewEditContactHistoryForm:previousNavigationPanel
                                :editCloseFollowUpConfirmDialog"

                        disableOnAjax="true"

                        oncomplete="handlePreviousNavigationComplete(args)">

                        <h:graphicImage
                            library="images"

                            name="#{independentContactHistoryController
                                    .viewEditFollowUpListBean.diablePrev
                                    ? 'common/button-previous-disabled.png'
                                    : 'common/button-previous.png'}"

                            alt="Previous contact history"

                            styleClass="history-navigation-image"/>

                    </p:commandLink>

                </h:panelGroup>

            </div>

            <!-- Remaining form content -->

        </h:form>

    </div>


    <h:outputScript target="body">

        function handlePreviousNavigationComplete(args) {

            setDatepicker();

            if (args) {

                if (args.dataModified === true) {

                    showConfirmationContactHistoryPopup(
                        'viewEditPrevFollowUpConfirmDialog',
                        400
                    );
                }
            }
        }

    </h:outputScript>

</ui:define>
```

---

# Complete final Java change

Add this method to:

```java
IndependentContactHistoryController
```

```java
import org.primefaces.PrimeFaces;

public void navigatePreviousContactHistory() {

    /*
     * Reuse the existing JSF 2 / RichFaces business logic.
     * Do not duplicate that logic here.
     */
    getContactHistoryForNoteIDPreNext("prev");

    /*
     * Send the latest server-side value to the PrimeFaces
     * oncomplete JavaScript callback.
     */
    PrimeFaces.current()
            .ajax()
            .addCallbackParam(
                    "dataModified",
                    isDataModified()
            );
}
```

When your getter is named differently, use the existing getter:

```java
PrimeFaces.current()
        .ajax()
        .addCallbackParam(
                "dataModified",
                getDataModified()
        );
```

---

# Line-by-line migration mapping

| Existing migration line           | Final change                                |
| --------------------------------- | ------------------------------------------- |
| `<p:commandButton>`               | Change to `<p:commandLink>`                 |
| Two command components            | Replace with one command component          |
| `image="..."`                     | Remove completely                           |
| Normal PNG                        | Put inside `h:graphicImage`                 |
| Disabled PNG                      | Select dynamically inside `h:graphicImage`  |
| `rendered="#{bean.diablePrev}"`   | Remove                                      |
| `rendered="#{!bean.diablePrev}"`  | Remove                                      |
| `disabled="#{bean.diablePrev}"`   | Keep                                        |
| `onclick="this.disabled=true;"`   | Remove                                      |
| `disableOnAjax="true"`            | Add or keep                                 |
| `action="#{bean.method('prev')}"` | Change to wrapper method                    |
| `process="@form"`                 | Keep initially                              |
| `update="..."`                    | Keep valid targets and add navigation panel |
| EL check in `oncomplete`          | Replace with callback `args`                |
| `styleClass="secondary"`          | Replace with image-link CSS class           |
| Hardcoded `/resources/...`        | Use `library` and `name`                    |

---

# Expected behaviour after migration

## When `diablePrev == false`

```text
button-previous.png displayed
Button is clickable
Action method executes
Ajax updates form sections
```

## When `diablePrev == true`

```text
button-previous-disabled.png displayed
Command link is disabled
Action method does not execute
```

## While Ajax is running

```text
PrimeFaces temporarily disables the link
Duplicate clicks are prevented
After completion, PrimeFaces restores the correct state
```

---

# Final checks

Verify these five items:

```text
1. No image attribute exists on p:commandLink or p:commandButton.
2. Only one previous-arrow command component remains.
3. Images exist under resources/images/common.
4. previousNavigationPanel is included in update.
5. Confirmation-dialog absolute ID matches its actual location.
```

The most important final component is:

```xhtml
<p:commandLink
    disabled="#{independentContactHistoryController
                .viewEditFollowUpListBean.diablePrev}">

    <h:graphicImage
        library="images"
        name="#{independentContactHistoryController
                .viewEditFollowUpListBean.diablePrev
                ? 'common/button-previous-disabled.png'
                : 'common/button-previous.png'}"/>

</p:commandLink>
```

[1]: https://primefaces.github.io/primefaces/vdldoc/p/commandLink.html "commandLink (PrimeFaces VDL Documentation)
            "
[2]: https://primefaces.github.io/primefaces/vdldoc/p/tld-summary.html "p (PrimeFaces VDL Documentation)
            "
[3]: https://jakarta.ee/specifications/faces/4.0/vdldoc/h/graphicimage "graphicImage (Jakarta Faces 4.0.0 VDL Documentation)
			"
[4]: https://primefaces.github.io/primefaces/jsdocs/modules/src_PrimeFaces.PrimeFaces.ajax.html?utm_source=chatgpt.com "ajax | PrimeFaces JavaScript API Docs"
