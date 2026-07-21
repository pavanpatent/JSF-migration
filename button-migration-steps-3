Your **Previous image is now working**. The remaining issue is that the **Next button is still a `p:commandButton` with an unsupported `image` attribute**.

Use `h:commandButton` for both arrows. Do not change the existing popup layout or add new grid CSS. Jakarta Faces supports `image` on `h:commandButton`; PrimeFaces `p:commandButton` does not provide that attribute. ([PrimeFaces][1])

## Step 1: Delete the old four buttons

Delete:

* Disabled Previous `p:commandButton`
* Enabled Previous `p:commandButton`
* Disabled Next `p:commandButton`
* Enabled Next `p:commandButton`

## Step 2: Paste this Previous button

```xhtml
<h:panelGroup id="previousNavigationPanel">

    <h:commandButton
        id="prvImg"
        styleClass="secondary"
        disabled="#{independentContactHistoryController
                    .viewEditFollowUpListBean.diablePrev}"
        image="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diablePrev
                 ? resource['images:common/button-previous-disabled.png']
                 : resource['images:common/button-previous.png']}"
        action="#{independentContactHistoryController
                  .getContactHistoryForNoteIDPreNext('prev')}"
        alt="Previous"
        title="Previous">

        <p:ajax
            process="@form"
            update=":viewEditContactHistoryForm:editFollowupNoteMainID
                    :viewEditContactHistoryForm:contactViewEditSaveCloseButtonsPanelID
                    :editCloseFollowUpConfirmDialog"
            oncomplete="setDatepicker();
                        if (#{independentContactHistoryController.dataModified == 'false'}) {
                        } else {
                            showConfirmationContactHistoryPopup(
                                'viewEditPrevFollowUpConfirmDialog',
                                400
                            );
                        }"/>

    </h:commandButton>

</h:panelGroup>
```

## Step 3: Paste this Next button

Place it where your existing Next button currently exists:

```xhtml
<h:panelGroup id="nextNavigationPanel">

    <h:commandButton
        id="nextImg"
        styleClass="secondary"
        disabled="#{independentContactHistoryController
                    .viewEditFollowUpListBean.diableNext}"
        image="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diableNext
                 ? resource['images:common/button-next-disabled.png']
                 : resource['images:common/button-next.png']}"
        action="#{independentContactHistoryController
                  .getContactHistoryForNoteIDPreNext('next')}"
        alt="Next"
        title="Next">

        <p:ajax
            process="@form"
            update=":viewEditContactHistoryForm:editFollowupNoteMainID
                    :viewEditContactHistoryForm:contactViewEditSaveCloseButtonsPanelID
                    :editCloseFollowUpConfirmDialog"
            oncomplete="setDatepicker();
                        if (#{independentContactHistoryController.dataModified == 'false'}) {
                        } else {
                            showConfirmationContactHistoryPopup(
                                'viewEditNextFollowUpConfirmDialog',
                                400
                            );
                        }"/>

    </h:commandButton>

</h:panelGroup>
```

Use your actual Next disabled property if its name is not `diableNext`.

## Step 4: Do not change these items

Keep:

```xhtml
styleClass="secondary"
process="@form"
```

Remove these old attributes completely:

```xhtml
rendered="..."
onclick="this.disabled=true;"
image="/resources/images/..."
```

Do not use:

```xhtml
<p:commandButton image="...">
```

## Step 5: Verify these four files

```text
src/main/webapp/resources/images/common/button-previous.png
src/main/webapp/resources/images/common/button-previous-disabled.png
src/main/webapp/resources/images/common/button-next.png
src/main/webapp/resources/images/common/button-next-disabled.png
```

## Final structure

Keep your existing positions:

```xhtml
<!-- Left side -->
<h:panelGroup id="previousNavigationPanel">
    <!-- Previous h:commandButton -->
</h:panelGroup>

<!-- Existing textarea stays unchanged -->

<!-- Right side -->
<h:panelGroup id="nextNavigationPanel">
    <!-- Next h:commandButton -->
</h:panelGroup>
```

The essential fix is converting the **Next button exactly the same way you converted the Previous button**.

[1]: https://primefaces.github.io/primefaces/vdldoc/primefaces/commandButton.html?utm_source=chatgpt.com "commandButton (PrimeFaces VDL Documentation)"
