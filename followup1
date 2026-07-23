Based on the first 110 lines in your screenshots, the main migration work is:

* Change old JSF namespaces to Jakarta Faces namespaces.
* Remove RichFaces `a4j`.
* Remove manually loaded jQuery 1.6.2 and jQuery UI 1.8.16.
* Replace the jQuery date picker with `<p:datePicker>`.
* Replace `c:if` with the JSF `rendered` attribute.
* Remove `<f:verbatim>`.
* Keep standard `<h:...>` tags where they already work.
* Use PrimeFaces for enhanced inputs, Ajax buttons, messages, and date pickers.

PrimeFaces 15 officially supports Jakarta Faces 4 using the Jakarta-classified dependency and the `xmlns:p="primefaces"` namespace. Jakarta Faces 4 uses namespaces such as `jakarta.faces.html`, `jakarta.faces.core`, and `jakarta.faces.facelets`. ([GitHub][1])

# Migrated JSF 4 + PrimeFaces 15 XHTML

```xhtml
<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:ui="jakarta.faces.facelets"
      xmlns:f="jakarta.faces.core"
      xmlns:h="jakarta.faces.html"
      xmlns:p="primefaces"
      xmlns:mot="http://www.midofficetools.bns.com/facelet/components">

<ui:composition
        template="/clientAccount/ClientAccountTemplate.xhtml">

    <!-- =========================================================
         Client Account Search navigation link
         ========================================================= -->
    <ui:define name="clientacctSearchLinkDef">

        <h:commandLink
                id="clientacctSearchLink"
                value="#{tBundle.CLIENTACCT_SEARCH_LINK}"
                actionListener="#{clientAccountNavBean.handleClick}"
                action="#{clientAccountNavBean.displayView}"/>

    </ui:define>


    <!-- =========================================================
         Client Account Follow-up navigation link
         ========================================================= -->
    <ui:define name="clientacctFollowUpLinkDef">

        <h:commandLink
                id="clientacctFollowUpLink"
                value="#{tBundle.CLIENTACCT_FOLLOW_UP_LINK}"
                action="#{followUpListController.getFollowUpList}"
                actionListener="#{clientAccountNavBean.handleClick}"
                styleClass="selected"/>

    </ui:define>


    <!-- =========================================================
         Main content
         ========================================================= -->
    <ui:define name="main-content">

        <!--
            Do not load jQuery manually here.

            REMOVE:
            jquery-1.6.2.min.js
            jquery-ui-1.8.16.custom.min.js
            jquery-ui-1.8.16.custom.css
        -->

        <!--
            Do not add another h:body here when the template already
            contains h:body.
        -->

        <h:form id="followUpForm">

            <!-- Replaces f:verbatim -->
            <h2>
                <h:outputText
                        value="#{tBundle.CLIENT_FOLLOW_UP_HEADER}"/>
            </h2>


            <!-- =================================================
                 Error messages
                 ================================================= -->
            <h:panelGroup
                    id="followUpMainErrorsSection"
                    layout="block">

                <!--
                    Recommended when errors are added as FacesMessage.
                -->
                <p:messages
                        id="followUpMessages"
                        showSummary="true"
                        showDetail="true"
                        closable="true"/>

                <!--
                    Keep this instead of p:messages only when the
                    custom component has been migrated to Jakarta:

                    <mot:showErrors/>
                -->

            </h:panelGroup>


            <!-- =================================================
                 Success message

                 Replaces:
                 <c:if test="...">
                 ================================================= -->
            <h:panelGroup
                    id="followUpMainSuccessSection"
                    layout="block"
                    styleClass="ewa-bucket"
                    rendered="#{
                        followUpListController
                            .followUpListBean
                            .successFlag
                    }">

                <ul class="success">
                    <li>
                        <h:outputText
                                value="#{
                                    followUpListController
                                        .followUpListBean
                                        .message
                                }"/>
                    </li>
                </ul>

            </h:panelGroup>


            <!-- =================================================
                 Follow-up search criteria
                 ================================================= -->
            <h:panelGroup
                    id="ccmFollowUp"
                    layout="block"
                    styleClass="block">


                <!-- Your dropdown code comes here -->



                <!-- =================================================
                     CSR LDAP number
                     ================================================= -->
                <h:panelGroup
                        id="csrLdapNumberDiv"
                        layout="block">

                    <h:panelGroup
                            layout="block"
                            styleClass="data data-title data4column">

                        <h:outputText
                                value="#{
                                    tBundle.FOLLOW_UP_USER_LDAP_LABEL
                                }#{tBundle.COLON_LABEL}"/>

                    </h:panelGroup>


                    <h:panelGroup
                            layout="block"
                            styleClass="data">

                        <p:inputText
                                id="csrLdapNumber"
                                value="#{
                                    followUpListController
                                        .followUpListBean
                                        .csrLdapId
                                }"/>

                    </h:panelGroup>

                    <br clear="all"/>

                    <f:event
                            type="preRenderComponent"
                            listener="#{
                                applicationAccessBean
                                    .checkComponentAccess
                            }"/>

                </h:panelGroup>


                <!-- =================================================
                     Follow-up date-range heading
                     ================================================= -->
                <h:panelGroup
                        layout="block"
                        styleClass="data data-title data4column">

                    <h:outputText
                            value="#{
                                tBundle.FOLLOW_UP_DATE_RANGE_LABEL
                            }#{tBundle.COLON_LABEL}"/>

                    <br/>

                </h:panelGroup>


                <!-- =================================================
                     From date
                     ================================================= -->
                <h:panelGroup
                        layout="block"
                        styleClass="data">

                    <h:outputText
                            value="#{
                                tBundle.FOLLOW_UP_FROM
                            }#{tBundle.COLON_LABEL}"/>

                    <br/>

                    <span class="note">
                        <h:outputText value="#{tBundle.DATE_FORMAT}"/>
                    </span>

                    <br/>

                    <p:datePicker
                            id="fromDate"
                            value="#{
                                followUpListController
                                    .followUpListBean
                                    .fromDate
                            }"
                            pattern="MM/dd/yyyy"
                            maxlength="10"
                            showIcon="true"
                            mask="99/99/9999"
                            monthNavigator="true"
                            yearNavigator="true"
                            inputStyleClass="followup-date"/>

                </h:panelGroup>


                <!-- =================================================
                     To date
                     ================================================= -->
                <h:panelGroup
                        layout="block"
                        styleClass="data">

                    <h:outputText
                            value="#{
                                tBundle.FOLLOW_UP_TO
                            }#{tBundle.COLON_LABEL}"/>

                    <br/>

                    <span class="note">
                        <h:outputText value="#{tBundle.DATE_FORMAT}"/>
                    </span>

                    <br/>

                    <p:datePicker
                            id="toDate"
                            value="#{
                                followUpListController
                                    .followUpListBean
                                    .toDate
                            }"
                            pattern="MM/dd/yyyy"
                            maxlength="10"
                            showIcon="true"
                            mask="99/99/9999"
                            monthNavigator="true"
                            yearNavigator="true"
                            inputStyleClass="followup-date"/>

                </h:panelGroup>

                <br clear="all"/>


                <!-- =================================================
                     Completed checkbox heading
                     ================================================= -->
                <h:panelGroup
                        layout="block"
                        styleClass="data data-title data4column">

                    <h:outputText
                            value="#{
                                tBundle.FOLLOW_UP_COMPLETED_LABEL
                            }#{tBundle.COLON_LABEL}"/>

                </h:panelGroup>


                <!-- =================================================
                     Completed checkbox
                     ================================================= -->
                <h:panelGroup
                        layout="block"
                        styleClass="data">

                    <p:selectBooleanCheckbox
                            id="client"
                            value="#{
                                followUpListController
                                    .followUpListBean
                                    .completed
                            }"/>

                </h:panelGroup>


                <!-- =================================================
                     Button section
                     ================================================= -->
                <h:panelGroup
                        id="followUpButtonSection"
                        layout="block">

                    <p:commandButton
                            id="followUpListResetButton"
                            value="#{tBundle.FOLLOW_UP_RESET}"
                            styleClass="secondary"
                            style="float:right"
                            action="#{
                                followUpListController
                                    .resetFollowUpSearch
                            }"
                            process="@this"
                            update="
                                ccmFollowUp
                                followUpMainErrorsSection
                                followUpMainSuccessSection
                            "
                            resetValues="true"/>


                    <p:commandButton
                            id="followUpListSearchUpButton"
                            value="#{tBundle.FOLLOW_UP_SEARCH}"
                            style="float:right"
                            action="#{
                                followUpListController
                                    .getFollowUpList
                            }"
                            process="ccmFollowUp"
                            update="
                                followUpMainErrorsSection
                                followUpMainSuccessSection
                                ccmFollowUp
                            "/>

                </h:panelGroup>

            </h:panelGroup>

        </h:form>

    </ui:define>

</ui:composition>

</html>
```

PrimeFaces provides `p:datePicker` as its date-selection component. `p:commandButton` supports Ajax processing through `process`, partial rendering through `update`, and component-state clearing through `resetValues`. ([PrimeFaces][2])

# Important corrections from your old page

## 1. Remove the old jQuery files

Delete these lines:

```xhtml
<script type="text/javascript"
        src="../resources/js/jquery/jquery-1.6.2.min.js">
</script>

<script type="text/javascript"
        src="../resources/js/jquery/jquery-ui-1.8.16.custom.min.js">
</script>

<link rel="stylesheet"
      type="text/css"
      href="../resources/css/jquery-ui-1.8.16.custom.css"/>
```

They were probably being used for this:

```xhtml
<h:inputText
        id="fromDate"
        styleClass="datepicker"/>
```

After migration, use:

```xhtml
<p:datePicker
        id="fromDate"
        value="#{followUpListController.followUpListBean.fromDate}"
        pattern="MM/dd/yyyy"
        showIcon="true"/>
```

Do not initialize it using:

```javascript
$(".datepicker").datepicker();
```

PrimeFaces manages its own date-picker widget and Ajax lifecycle.

---

## 2. Do not replace every `h:` tag

These are still completely valid in JSF 4:

```xhtml
<h:form>
<h:panelGroup>
<h:outputText>
<h:commandLink>
```

Use PrimeFaces where additional functionality is useful:

```xhtml
<p:datePicker>
<p:commandButton>
<p:messages>
<p:selectBooleanCheckbox>
<p:inputText>
```

Therefore:

```text
h:panelGroup does not have to become p:outputPanel
h:outputText does not have to become another PrimeFaces component
h:form does not have to change
```

---

## 3. Replace `c:if` with `rendered`

Your current code:

```xhtml
<c:if test="#{
    followUpListController.followUpListBean.successFlag
}">
    <h:panelGroup>
        ...
    </h:panelGroup>
</c:if>
```

Use:

```xhtml
<h:panelGroup
        id="followUpMainSuccessSection"
        layout="block"
        rendered="#{
            followUpListController.followUpListBean.successFlag
        }">

    ...

</h:panelGroup>
```

This is safer for JSF Ajax because the component remains part of the JSF view definition.

When `rendered="false"`, the component itself will not produce HTML. Therefore, when you need to update it independently through Ajax, wrap it:

```xhtml
<h:panelGroup
        id="successSectionWrapper"
        layout="block">

    <h:panelGroup
            id="followUpMainSuccessSection"
            rendered="#{
                followUpListController
                    .followUpListBean
                    .successFlag
            }">

        ...

    </h:panelGroup>

</h:panelGroup>
```

Then update the wrapper:

```xhtml
update="successSectionWrapper"
```

This wrapper approach is preferable for your page.

---

# Recommended corrected success section

Use this version in the final page:

```xhtml
<h:panelGroup
        id="successSectionWrapper"
        layout="block">

    <h:panelGroup
            id="followUpMainSuccessSection"
            layout="block"
            styleClass="ewa-bucket"
            rendered="#{
                followUpListController
                    .followUpListBean
                    .successFlag
            }">

        <ul class="success">
            <li>
                <h:outputText
                        value="#{
                            followUpListController
                                .followUpListBean
                                .message
                        }"/>
            </li>
        </ul>

    </h:panelGroup>

</h:panelGroup>
```

And buttons should update:

```xhtml
update="
    followUpMainErrorsSection
    successSectionWrapper
    ccmFollowUp
"
```

---

# Date field Java type

Check the current Java types of:

```java
followUpListBean.fromDate
followUpListBean.toDate
```

The recommended JSF 4 design is:

```java
import java.time.LocalDate;

public class FollowUpListBean {

    private LocalDate fromDate;
    private LocalDate toDate;

    public LocalDate getFromDate() {
        return fromDate;
    }

    public void setFromDate(LocalDate fromDate) {
        this.fromDate = fromDate;
    }

    public LocalDate getToDate() {
        return toDate;
    }

    public void setToDate(LocalDate toDate) {
        this.toDate = toDate;
    }
}
```

Then the XHTML can directly use:

```xhtml
<p:datePicker
        value="#{followUpListController.followUpListBean.fromDate}"
        pattern="MM/dd/yyyy"/>
```

Change the pattern when your application uses another format:

```text
MM/dd/yyyy    07/23/2026
dd/MM/yyyy    23/07/2026
yyyy-MM-dd    2026-07-23
```

---

# Reset method required

Your original reset button appears to have no action method:

```xhtml
<h:commandButton
        id="followUpListResetButton"
        value="#{tBundle.FOLLOW_UP_RESET}"/>
```

That does not reliably reset the server-side bean. Add an explicit method:

```java
public void resetFollowUpSearch() {

    followUpListBean.setCsrLdapId(null);
    followUpListBean.setFromDate(null);
    followUpListBean.setToDate(null);
    followUpListBean.setCompleted(false);

    followUpListBean.setSuccessFlag(false);
    followUpListBean.setMessage(null);
}
```

Then call it:

```xhtml
<p:commandButton
        id="followUpListResetButton"
        value="#{tBundle.FOLLOW_UP_RESET}"
        action="#{followUpListController.resetFollowUpSearch}"
        process="@this"
        update="
            ccmFollowUp
            followUpMainErrorsSection
            successSectionWrapper
        "
        resetValues="true"/>
```

PrimeFaces documents `resetValues` specifically for clearing local component values, which is especially important after validation errors. ([PrimeFaces][3])

# Two items that need separate verification

### `mot:showErrors`

This is your internal custom component:

```xhtml
xmlns:mot="http://www.midofficetools.bns.com/facelet/components"
```

It can remain only when its Java implementation has also been migrated from:

```java
javax.faces.*
```

to:

```java
jakarta.faces.*
```

When it is only displaying `FacesMessage` objects, replace it with:

```xhtml
<p:messages
        id="followUpMessages"
        showSummary="true"
        showDetail="true"/>
```

### `applicationAccessBean.checkComponentAccess`

The XHTML remains:

```xhtml
<f:event
        type="preRenderComponent"
        listener="#{
            applicationAccessBean.checkComponentAccess
        }"/>
```

But any Java import such as:

```java
import javax.faces.event.ComponentSystemEvent;
```

must become:

```java
import jakarta.faces.event.ComponentSystemEvent;
```

The next migration step should be to examine the **dropdown section and the result table after line 110**, because that area may contain actual RichFaces tags such as `rich:dataTable`, `rich:column`, `a4j:ajax`, `rich:popupPanel`, or `rich:calendar` that require direct PrimeFaces replacements.

[1]: https://github.com/primefaces/primefaces "GitHub - primefaces/primefaces: Ultimate Component Suite for Jakarta Faces · GitHub"
[2]: https://primefaces.github.io/primefaces/vdldoc/primefaces/datePicker.html "datePicker (PrimeFaces VDL Documentation)
            "
[3]: https://primefaces.github.io/primefaces/vdldoc/primefaces/commandButton.html?utm_source=chatgpt.com "commandButton (PrimeFaces VDL Documentation)"
