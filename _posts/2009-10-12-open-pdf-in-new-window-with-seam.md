---
title: Open PDF in New Window with Seam
description: A simple method to open PDFs in a new window while displaying validation errors in the original window. The PDF window will not open if there are validation or other messages.
tags: development seam
---

For a recent Seam development project, I had the task of creating a PDF report from some user input and displaying the result in a new window.

Initially, I used a `<h:commandLink/>` with `target="_blank"`. This worked well until the user entered criteria that failed validation. The new window was still created, and the validation messages were shown in the new window. This was not a good solution, so after some research and experimentation I came up a solution that meets the following criteria:

*   Validation messages are shown in original window.
*   A new window is not created when there are validation messages.
*   A new window is not created when the entered criteria returns zero (0) results.

### The Action Class

For this example, the action class is very simple:

```
public String getReportURL() {
  return "/pdfpopup/reports/reportDisplay.seam";
}
// Getters and setters omitted
public List<String> getReportData() {
  return reportData;
}
public void doSearch() {
  // This is where the work would actually happen
  reportData = new ArrayList<String>();
  for (int i = 0; i < selectOneValue; i++) {
    reportData.add("Report Row " + (i + 1));
  }
  if (reportData.isEmpty()) {
    facesMessages.add("Report will be empty, not popping up");
  }
}
```

I added the `getReportURL()` method to the action because there may be occasions where the report URL differs based on criteria. In the application I worked on, we had to support PDF and Excel reports, so the method returned the correct URL.

The action simply takes the criteria specified on the JSF page and populates a list of results based on that criteria. The PDF generation page will access the list later.

### The JSF Page

The JSF page looks like this:

```
<ui:define name="body">
  <script type="text/javascript">
  //<![CDATA[
  function showReport(conversationId) {
    if (document.getElementById("messages") != null) {
      return;
    }
    var reportWin = window.open('#{reportAction.reportURL}' + '?cid=' + conversationId);
    if (!reportWin) {
      alert("Could not open the report window. Please disable popup blocking for this website and try again.");
    }
  }
  // ]]>
  </script>
  <h:form id="generateReport">
    <s:validateAll>
      <h:panelGrid width="100%" columns="1" style="text-align: center; font-weight:bold; font-size: 12px">
        <h:outputText value="Report Query"/>
      </h:panelGrid>
      <h:panelGrid columns="2" border="0" frame="none" style="padding-top:30px;">
        <h:outputLabel for="selectOneValue">Number of Output Rows</h:outputLabel>
        <h:selectOneMenu id="selectOneValue" value="#{reportAction.selectOneValue}" required="true">
          <f:selectItem itemLabel="Zero" itemValue="0"/>
          <f:selectItem itemLabel="One" itemValue="1"/>
          <f:selectItem itemLabel="Two" itemValue="2"/>
          <f:selectItem itemLabel="Three" itemValue="3"/>
          <f:selectItem itemLabel="Four" itemValue="4"/>
        </h:selectOneMenu>
        <h:outputLabel for="startDate">Report Period:</h:outputLabel>
        <h:panelGroup>
          <rich:calendar id="startDate" enableManualInput="true" value="#{reportAction.startDate}" showWeeksBar="false" datePattern="MM/dd/yyyy" immediate="true" required="true" label="Report Start Date"/>
          <h:outputLabel for="endDate">through</h:outputLabel>
          <rich:calendar id="endDate" enableManualInput="true" value="#{reportAction.endDate}" showWeeksBar="false" datePattern="MM/dd/yyyy" immediate="true" required="true" label="Report End Date"/>
        </h:panelGroup>
      <a4j:commandButton id="getReportLink" action="#{reportAction.doSearch}" value="Get Report" oncomplete="showReport('#{conversation.id}')"/>
     </h:panelGrid>
    </s:validateAll>
  </h:form>
</ui:define>
```

This is really the guts of the operation. When the `commandButton` is clicked, the `doSearch()` method will be executed. Once that call is complete, the `oncomplete` handler will be called.

The page template is configured so that any `FacesMessages` will be displayed in an element with id “`messages`”. If this element is present, we assume that either a validation error or no data message is being displayed and do not show the PDF. If the element is not present, everything is OK and we open the new window with the URL for the PDF. Note that it’s crucial to pass the conversation ID along so that the data loaded by the action will be visible to the page creating the PDF.

### The PDF

For completeness, here’s the sample page I’m using to create a PDF:

```
<p:document xmlns:p="http://jboss.com/products/seam/pdf" xmlns:ui="http://java.sun.com/jsf/facelets" xmlns:f="http://java.sun.com/jsf/core" title="Sample PDF Report">
  <f:facet name="header">
    <p:font size="12">
      <p:header borderWidthTop="0" borderWidthBottom="0.4" alignment="center">
        <p:paragraph indentationLeft="50">REPORT HEADER</p:paragraph>
      </p:header>
      <p:footer borderWidthTop="1" borderColorTop="black" borderWidthBottom="0" alignment="center"> [<p:pageNumber/>] </p:footer>
    </p:font>
  </f:facet>
  <ui:repeat var="item" value="#{reportAction.reportData}">
    <p:font size="8">
      <p:table columns="1" headerRows="1" widthPercentage="100">
        <p:font style="bold">
          <p:cell verticalAlignment="bottom">Report Data</p:cell>
        </p:font>
        <p:cell>#{item}</p:cell>
      </p:table>
    </p:font>
  </ui:repeat>
</p:document>
```

### Full Sample Code

You can view and download the full sample code for this solution from [here](https://github.com/iamthechad/pdfpopup).
