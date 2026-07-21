## Final click fix

The left arrow will not click when `diablePrev=true`; Jakarta Faces renders it as disabled and excludes it from submission. That is expected. ([Jakarta EE][1])

For the enabled arrow, make these exact changes:

1. Use no-argument controller methods.
2. Change `process="@form"` to `process="@this"` so other form validations cannot block navigation.
3. Add `immediate="true"` so the navigation action runs early in the JSF lifecycle. ([PrimeFaces][2])

### 1. Add these methods to the controller

```java
public String previousContactHistory() {

    getContactHistoryForNoteIDPreNext("prev");

    return null;
}

public String nextContactHistory() {

    getContactHistoryForNoteIDPreNext("next");

    return null;
}
```

Keep your existing method unchanged:

```java
getContactHistoryForNoteIDPreNext(String direction)
```

---

### 2. Final Previous button

```xhtml
<h:panelGroup id="previousNavigationPanel">

    <h:commandButton
        id="prvImg"
        styleClass="secondary"

        disabled="#{independentContactHistoryController
                    .viewEditFollowUpListBean.diablePrev}"

        immediate="true"

        action="#{independentContactHistoryController
                  .previousContactHistory}"

        image="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diablePrev
                 ? resource['images:common/button-previous-disabled.png']
                 : resource['images:common/button-previous.png']}"

        alt="Previous"
        title="Previous">

        <p:ajax
            process="@this"

            update=":viewEditContactHistoryForm:editFollowupNoteMainID"

            oncomplete="setDatepicker();"/>

    </h:commandButton>

</h:panelGroup>
```

---

### 3. Final Next button

```xhtml
<h:panelGroup id="nextNavigationPanel">

    <h:commandButton
        id="nextImg"
        styleClass="secondary"

        disabled="#{independentContactHistoryController
                    .viewEditFollowUpListBean.diableNext}"

        immediate="true"

        action="#{independentContactHistoryController
                  .nextContactHistory}"

        image="#{independentContactHistoryController
                 .viewEditFollowUpListBean.diableNext
                 ? resource['images:common/button-next-disabled.png']
                 : resource['images:common/button-next.png']}"

        alt="Next"
        title="Next">

        <p:ajax
            process="@this"

            update=":viewEditContactHistoryForm:editFollowupNoteMainID"

            oncomplete="setDatepicker();"/>

    </h:commandButton>

</h:panelGroup>
```

## Important changes

Replace:

```xhtml
process="@form"
```

with:

```xhtml
process="@this"
```

Add:

```xhtml
immediate="true"
```

Replace:

```xhtml
action="#{independentContactHistoryController
          .getContactHistoryForNoteIDPreNext('next')}"
```

with:

```xhtml
action="#{independentContactHistoryController.nextContactHistory}"
```

Do the same for Previous.

For the first test, update only:

```xhtml
update=":viewEditContactHistoryForm:editFollowupNoteMainID"
```

This avoids an invalid confirmation-dialog component ID blocking the Ajax response. PrimeFaces `process` identifies what participates in the request, and `update` identifies what is refreshed afterward. ([PrimeFaces][2])

Expected behavior:

```text
diablePrev = true  → Previous arrow cannot click
diablePrev = false → Previous action executes

diableNext = true  → Next arrow cannot click
diableNext = false → Next action executes
```

[1]: https://jakarta.ee/specifications/faces/4.0/vdldoc/h/commandbutton "commandButton (Jakarta Faces 4.0.0 VDL Documentation)
			"
[2]: https://primefaces.github.io/primefaces/vdldoc/p/ajax.html "ajax (PrimeFaces VDL Documentation)
            "
