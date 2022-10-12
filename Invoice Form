/**
 *  @author William Brewer <admin@atlantaadminsolutions.com>
 *
 *  @global
 *  @default
 */
let disableBillSubmitButton = true

/**
 *  Collect invoice data
 *  @param {Object} e 
 *  @param {Object} e.formInput
 *  @param {String} e.clientPlatform
 *  @param {Object} e.parameters
 *  @param {Object} e.commonEventObject
 *  @param {String} e.hostApp
 *  @param {Object} e.gmail
 *  @param {String} e.accessToken
 *  @param {Object} e.formInputs
 *  @param {Object} e.messageMetadata
 *  @return {Card}
*/
function gotoBill(e) {
  console.log("gotoBill(e) called")

  let card
  try {
    PropertiesService.getScriptProperties().deleteAllProperties()

    let submitBillAction = CardService.newAction().setFunctionName('submitButtonClicked');
    let splitAction = CardService.newAction().setFunctionName('splitButtonClicked');
    let submitBillBtn = CardService.newTextButton()
      .setText('Submit')
      .setOnClickAction(submitBillAction)

    let splitBtn = CardService.newTextButton()
      .setText('Split')
      .setOnClickAction(splitAction)

    let fixedFooter = CardService.newFixedFooter().setPrimaryButton(splitBtn).setSecondaryButton(submitBillBtn)

    card = CardService.newCardBuilder().setHeader(CardService.newCardHeader().setTitle("invoice Card")).setFixedFooter(fixedFooter)

    let inputSection = CardService.newCardSection()
    let vendorAction = CardService.newAction().setFunctionName('getVendors')
    let vendor = CardService.newTextInput().setFieldName('vendor').setTitle('Vendor').setSuggestionsAction(vendorAction)
    let currentMessage = getCurrentMessage(e)
    let receivedDate = getReceivedDate(currentMessage)
    let date = CardService.newDatePicker().setValueInMsSinceEpoch(new Date(receivedDate).getTime()).setFieldName("date")
    let ref = CardService.newTextInput().setFieldName('refNum').setTitle('Ref #')
    let dueDate = CardService.newDatePicker().setFieldName("dueDate").setTitle("Due Date")
    let propertyAction = CardService.newAction().setFunctionName('getProperties')
    let job = CardService.newTextInput().setFieldName('property').setTitle('Property').setSuggestionsAction(propertyAction)
    let memo = CardService.newTextInput().setFieldName('memo').setTitle('Memo').setMultiline(true)
    let attachments = getAttachment(currentMessage)
    let billAttachments = CardService.newSelectionInput().setType(CardService.SelectionInputType.RADIO_BUTTON)
      .setFieldName("attachments").addItem('No attachment', 'No attachment', true)
    let lineitem = CardService.newTextInput().setFieldName('lineitem').setTitle('Line Item ').setMultiline(true)
    let itemAmount = CardService.newTextInput().setFieldName('itemAmount').setTitle('Item Amount').setHint("USD")
    if (attachments.length == 0) billAttachments.addItem("Email body as attachment", 'Email body as attachment', true)
    if (attachments.length == 1) billAttachments.addItem(attachments[0].getName(), attachments[0].getName(), true)
    if (attachments.length > 1) {
      attachments.forEach(function (attachment, index) {
        if (attachment.getContentType() == 'image/jpeg' ||
          attachment.getContentType() == 'application/pdf' ||
          attachment.getContentType() == 'image/jpg'
        ) {
          billAttachments.addItem(attachment.getName(), attachment.getName(), true)
        }
      })
    }

    try { vendor.setValue(e.formInput.vendor) } catch { }
    try { ref.setValue(e.formInput.refNum) } catch { }
    try { dueDate.setValueInMsSinceEpoch(e.formInput.dueDate.msSinceEpoch) } catch { }
    try { job.setValue(e.formInput.property) } catch { }
    try { lineitem.setValue(e.formInput.lineitem) } catch { }
    try { itemAmount.setValue(e.formInput.itemAmount) } catch { }
    try { memo.setValue(e.formInput.memo) } catch { }

    inputSection.addWidget(CardService.newTextParagraph().setText("Attachments"))
      .addWidget(billAttachments)
      .addWidget(date)
      .addWidget(vendor)
      .addWidget(ref)
      .addWidget(dueDate)
      .addWidget(job)
      .addWidget(lineitem)
      .addWidget(itemAmount)
      .addWidget(memo)

    card.addSection(inputSection)
  } catch (e) {

    card = CardService.newCardBuilder()
    let section = CardService.newCardSection()
    let image = CardService.newImage().setImageUrl("data:image/jpeg;base64," + outOfScopeEmailImageBase64)
    section.addWidget(image)
    card.addSection(section)
  }
  return card.build()

}
/**
 *  Collect vendor,invoice number, due date  
 *  @param {Object} e object containing the service property card to build.
 *  @return {Card}
*/
function billDataCard() {

  let clickAction = CardService.newAction().setFunctionName('processFormData');
  let splitBtn = CardService.newTextButton()
    .setText('Continue')
    .setOnClickAction(clickAction)

  let fixedFooter = CardService.newFixedFooter().setPrimaryButton(splitBtn)
  let card = CardService.newCardBuilder().setFixedFooter(fixedFooter)
  let section = CardService.newCardSection().setHeader("invoice Data")
  let vendorAction = CardService.newAction().setFunctionName('getVendors')
  let vendor = CardService.newTextInput().setFieldName('vendor').setTitle('Vendor').setSuggestionsAction(vendorAction)
  let ref = CardService.newTextInput().setFieldName('refNum').setTitle('Ref #')
  let dueDate = CardService.newDatePicker().setFieldName("dueDate").setTitle("Due Date")

  section.addWidget(vendor)
    .addWidget(ref)
    .addWidget(dueDate)

  card.addSection(section)

  return card.build()

}
/**
 *  Collect property,lineitem, item amount 
 *  @param {Object} e object containing the service property card to build.
 *  @return {Card}
*/
function splitCard(e) {
  console.log("splitCard(e) called");
  let scriptProperties = PropertiesService.getScriptProperties()
  let lineitems = JSON.parse(scriptProperties.getProperty('lineitems'))
  let submitBillAction = CardService.newAction().setFunctionName('invoiceDetailCard');
  let submitBillBtn = CardService.newTextButton()
    .setText('Add To invoice')
    .setOnClickAction(submitBillAction)
    .setDisabled(disableBillSubmitButton)
  let addItemAction = CardService.newAction().setFunctionName('updateLineItemList');
  let addItemBtn = CardService.newTextButton()
    .setText('Add Line Item')
    .setOnClickAction(addItemAction)
  let fixedFooter = CardService.newFixedFooter().setPrimaryButton(addItemBtn).setSecondaryButton(submitBillBtn)
  let card = CardService.newCardBuilder().setHeader(CardService.newCardHeader().setTitle("Splits Card")).setFixedFooter(fixedFooter)
  let section = CardService.newCardSection().setHeader("Line Data")
  let itemSection = CardService.newCardSection().setHeader("Line Items").setCollapsible(false);
  let totalSection = CardService.newCardSection().setHeader("invoice Total")
  let invoiceTotal = 0;
  let propertyAction = CardService.newAction().setFunctionName('getProperties')
  let job = CardService.newTextInput().setFieldName('property').setTitle('Property').setSuggestionsAction(propertyAction)
  let lineitem = CardService.newTextInput().setFieldName('lineitem').setTitle('Line Item ').setMultiline(true)
  let itemAmount = CardService.newTextInput().setFieldName('itemAmount').setTitle('Item Amount').setHint("USD")

  try {job.setValue(e.formInput.property)} catch{}
  try {lineitem.setValue(e.formInput.lineitem)} catch{}
  try {itemAmount.setValue(e.formInput.itemAmount)} catch{}
    
  if (lineitems != null) {
    try {
      lineitems.forEach(function (item, index) {
        let lineAmount = parseFloat(item[1])
        invoiceTotal += lineAmount
        let _lineItem = item[0]
        let amount = "$" + (new Intl.NumberFormat({ style: 'currency', currency: 'USD' }).format(item[1]))
        let removeItem = CardService.newSelectionInput()
          .setType(CardService.SelectionInputType.CHECK_BOX)
          .setFieldName("checkbox_field")
          .addItem("Delete", index, false)
          .setOnChangeAction(CardService.newAction()
            .setFunctionName("updateLineItemList"));

        itemSection.addWidget(CardService.newDecoratedText().setText(_lineItem + "\n" + amount.padStart(60))
          .setTopLabel(item[2]).setWrapText(true));
        itemSection.addWidget(removeItem)
        itemSection.addWidget(CardService.newDivider())

      })
      invoiceTotal = "$" + (new Intl.NumberFormat({ style: 'currency', currency: 'USD' }).format(invoiceTotal))
      PropertiesService.getScriptProperties().deleteProperty('invoiceTotal').setProperty('invoiceTotal', JSON.stringify(invoiceTotal))
      totalSection.addWidget(CardService.newTextParagraph()
        .setText(invoiceTotal.padStart(60))
      )
    } catch (e) { }
  }
  if (lineitems != null) {
    card.addSection(totalSection)
  }
  section.addWidget(job)
    .addWidget(lineitem)
    .addWidget(itemAmount)
  card.addSection(section)
  if (lineitems != null) {
    card.addSection(itemSection)
  }
  return card.build()
}
/**
 *  Update lineitem details 
 *  @param {Object} e - event object
 *  @return {Card}
*/
function lineitemClicked(e){
  return updateLineitemCard(e)
}
function updateLineitemCard(e){
    let cardFooter1Button1Action1 = CardService.newAction()
        .setFunctionName('deleteLineitemClicked')
        .setParameters({});

    let cardFooter1Button1 = CardService.newTextButton()
        .setText('Delete Line item')
        .setOnClickAction(cardFooter1Button1Action1);

    let cardFooter1 = CardService.newFixedFooter()
        .setPrimaryButton(cardFooter1Button1);

    let cardSection1TextInput1 = CardService.newTextInput()
        .setFieldName('Property')
        .setTitle('Property')
        .setMultiline(false);

    let cardSection1TextInput2 = CardService.newTextInput()
        .setFieldName('amount')
        .setTitle('Amount')
        .setMultiline(false);

    let cardSection1TextInput3 = CardService.newTextInput()
        .setFieldName('lineitem')
        .setTitle('Line item')
        .setMultiline(false);

    let cardSection1 = CardService.newCardSection()
        .addWidget(cardSection1TextInput1)
        .addWidget(cardSection1TextInput2)
        .addWidget(cardSection1TextInput3);

    let card = CardService.newCardBuilder()
        .setFixedFooter(cardFooter1)
        .addSection(cardSection1)
        .build();
    return card;

}

/**
 *  Confirm Invoice details are correct 
 *  @param {Object} e - event object
 *  @return {Card}
*/
function invoiceDetailCard(e) {
  console.log("invoiceDetailCard(e) called")
  let invoice = PropertiesService.getScriptProperties().getProperties()

  let cardFooter1Button1Action1 = CardService.newAction()
    .setFunctionName('submitBill')

  let cardFooter1Button1 = CardService.newTextButton()
    .setText('Submit')
    .setDisabled(false)
    .setOnClickAction(cardFooter1Button1Action1);

  let cardFooter1 = CardService.newFixedFooter()
    .setPrimaryButton(cardFooter1Button1);

  let cardSection1 = CardService.newCardSection()
    .setHeader('Invoice details')

  let invoiceData = JSON.parse(invoice.invoiceData)

  for (let _invoiceData in invoiceData) {

    let decoratedText = CardService.newDecoratedText()

    if (_invoiceData == 'Vendor') {
      decoratedText
        .setText(invoiceData[_invoiceData])
        .setBottomLabel(_invoiceData)
    }
    if (_invoiceData == 'Invoice Number') {
      decoratedText
        .setText(invoiceData[_invoiceData])
        .setBottomLabel(_invoiceData)
    }
    if (_invoiceData == 'Due Date') {
      decoratedText
        .setText(ATL.dateFormat(new Date(invoiceData[_invoiceData])))
        .setBottomLabel(_invoiceData)
    }
    cardSection1.addWidget(decoratedText)
  }

  let cardSection1DecoratedText4 = CardService.newDecoratedText()
    .setText(JSON.parse(invoice.invoiceTotal))
    .setBottomLabel('Invoice Amount');

  let cardSection1DecoratedText5 = CardService.newDecoratedText()
    .setText(invoice.attachment)
    .setBottomLabel('Attachment')
    .setWrapText(true);


  cardSection1
    .addWidget(cardSection1DecoratedText4)
    .addWidget(cardSection1DecoratedText5)

  let cardSection2 = CardService.newCardSection()
    .setHeader('Lineitems')

  let decoratedText = CardService.newDecoratedText()
  let lineitems = ''
  let lineitemTotalByJob = 0
  let currentProperty

  JSON.parse(invoice.lineitems).forEach(lineitem => {

    if (currentProperty == undefined) {
      currentProperty = lineitem[2]
    }
    if (lineitem[2] == currentProperty) {
      lineitems += (lineitem[0] + "\n")
      lineitemTotalByJob += parseFloat(lineitem[1])
    } else {
      decoratedText
        .setTopLabel(currentProperty)
        .setText(lineitems)
        .setBottomLabel(lineitemTotalByJob)
        .setWrapText(true);

      cardSection2.addWidget(decoratedText)
      currentProperty = lineitem[2]
      lineitemTotalByJob = parseFloat(lineitem[1])
      lineitems = (lineitem[0] + "\n")
    }
  })

  decoratedText
    .setTopLabel(currentProperty)
    .setText(lineitems)
    .setBottomLabel(lineitemTotalByJob)
    .setWrapText(true);

  cardSection2.addWidget(decoratedText)

  let card = CardService.newCardBuilder()
    .setFixedFooter(cardFooter1)
    .addSection(cardSection1)
    .addSection(cardSection2)
    .build();
  return card;
}
/**
 *  Add and remove line items
 *  @param {Object} e - event object
 *  @return {Navigation}
*/
function updateLineItemList(e) {
  console.log("updateLineItemList(e) called")
  disableBillSubmitButton = false

  let lineItemList = (PropertiesService.getScriptProperties().getProperty("lineitems") != null) ? JSON.parse(PropertiesService.getScriptProperties().getProperty("lineitems")) : []

  if (e.formInputs.checkbox_field != null) {
    //pop lineitem from lineItemList
    let updatedLineItemList = []
    lineItemList.forEach(function (item, index) {
      if (index == parseInt(e.formInputs.checkbox_field)) return
      updatedLineItemList.push(item)
    })
    lineItemList = updatedLineItemList

  } else {
    //add new line item
    lineItemList.push([e.formInputs.lineitem[0], e.formInputs.itemAmount[0], e.formInputs.property[0]])
  }
  PropertiesService.getScriptProperties().deleteProperty('lineitems').setProperty('lineitems', JSON.stringify(lineItemList))

  let navigation = CardService.newNavigation().updateCard(splitCard(e))
  return CardService.newActionResponseBuilder().setNavigation(navigation).build()
}
/**
 *  Requires the following input feilds
 *  @param {Object} e - event object
 *  @param {String} e.formInput.vendor
 *  @param {String} e.formInput.refNum
 *  @param {String} e.formInput.dueDate
 *  @param {String} e.formInput.property
 *  @param {String} e.formInput.lineitem
 *  @param {String} e.formInput.itemAmount
 *  @return {Boolean}
*/
function hasInvoiceData(e) {
  console.log('hasInvoiceData(e) called')

  return (e.formInput.vendor != null && e.formInput.refNum != null && e.formInput.dueDate != null && e.formInput.property != null && e.formInput.lineitem != null && e.formInput.itemAmount != null)

}
/**
 *  Click Action
 *  @param {Object} e - event object
 *  @return {Card|Navigation} 
*/
function submitButtonClicked(e) {
  console.log('submitButtonClicked(e) called')

  let navigation = CardService.newNavigation().popCard().pushCard(gotoBill(e));
  if (hasInvoiceData(e)) {
    return submitBill(e)
  } else {
    return CardService.newActionResponseBuilder().setNavigation(navigation).build();
  }

}
/**
 * Create a row in table and create file in drive
 * @param {Event} e - event object
 * return {Navigation}
 */
function submitBill(e) {
  console.log("submitBill(e) called")
  let timestamp = (new Date().getMonth() + 1) + '/' + new Date().getDate() + '/' + new Date().getFullYear() + " " + new Date().toTimeString().split(' ')[0]
  let invoice = PropertiesService.getScriptProperties().getProperties()
  let invoiceRow
  let invoiceItemRow
  let navigation = CardService.newNavigation().popToRoot();
  let invoiceId;
  let fileData = addToDrive({ e: e, sheet: 'Invoices' })
  if (Object.keys(invoice).length != 0) {
    let lineitemTotalByJob = 0
    let property = undefined
    invoiceId = fileData.key
    JSON.parse(invoice.lineitems).forEach((element, index) => {
      if (property == undefined) {
        property = element[2]
      }

      if (element[2] == property) {
        lineitemTotalByJob += parseFloat(element[1])
      } else {

        invoiceRow = [
          invoiceId,
          timestamp,
          ATL.dateFormat(new Date(new Date().setTime(parseInt(JSON.parse(invoice.invoiceData)['Due Date']))), 'd'),
          property,
          JSON.parse(invoice.invoiceData)['Vendor'],
          JSON.parse(invoice.invoiceData)['Invoice Number'],
          lineitemTotalByJob,
          '',
          (fileData.type == 'image') ? fileData.filePath : '',
          (fileData.type == 'file') ? fileData.filePath : '',
          false,
          '',
          '',
          '0',
          Session.getActiveUser().getEmail(),
          false
        ]
        appendRowData([{ row: invoiceRow, sheet: 'Invoices' }])
        lineitemTotalByJob = 0
        property = element[2]
        lineitemTotalByJob = parseFloat(element[1])
      }

      invoiceItemRow = [
        ATL.getKey(),
        (index == 0) ? invoiceId : invoiceId = ATL.getKey(),
        '',
        element[0],//item
        JSON.parse(element[1]) //amount
        , ''//memo
      ]

      appendRowData([{ row: invoiceItemRow, sheet: 'Invoice Items' }])
    })

    invoiceRow = [
      invoiceId,
      timestamp,
      ATL.dateFormat(new Date(new Date().setTime(parseInt(JSON.parse(invoice.invoiceData)['Due Date']))), 'd'),
      property,
      JSON.parse(invoice.invoiceData)['Vendor'],
      JSON.parse(invoice.invoiceData)['Invoice Number'],
      lineitemTotalByJob,
      '',
      (fileData.type == 'image') ? fileData.filePath : '',
      (fileData.type == 'file') ? fileData.filePath : '',
      false,
      '',
      '',
      '0',
      Session.getActiveUser().getEmail(),
      false
    ]

    appendRowData([{ row: invoiceRow, sheet: 'Invoices' }])

  } else {
    invoiceId = ATL.getKey()
    invoiceRow = [
      invoiceId,
      timestamp,
      ATL.dateFormat(new Date(new Date().setTime(e.formInput.date.msSinceEpoch)), 'd'),
      e.formInput.property,
      e.formInput.vendor,
      e.formInput.refNum,
      e.formInput.itemAmount,
      '',
      (fileData.type == 'image') ? fileData.filePath : '',
      (fileData.type == 'file') ? fileData.filePath : '',
      false,
      '',
      '',
      '0',
      Session.getActiveUser().getEmail(),
      false
    ]

    invoiceItemRow = [
      ATL.getKey(),
      invoiceId,
      '',
      e.formInput.lineitem,
      e.formInput.itemAmount,
      e.formInput.memo
    ]
    appendRowData([{ row: invoiceRow, sheet: 'Invoices' }, { row: invoiceItemRow, sheet: 'Invoice Items' }])
  }
  //archiveMessage(e)
  return CardService.newActionResponseBuilder().setNavigation(navigation).build();
}
/**
  * Append data to Table
  * @param {Array} data
  */
function appendRowData(data) {
  console.log('appendRowData(data) Called')
  if (arguments.length < 1) throw 'No Argument'
  data.forEach(rowData => {
    sheets[rowData.sheet].sheet.appendRow(rowData.row)
  })
}

/**
 * add to drive
 *  @Param {Object} data - event object
 *  @Return {Object} File Attachment type and filepath
 */
function addToDrive(data) {
  console.log("addToDrive(e) Called")

  let e = data.e
  let message = getCurrentMessage(e)
  let attachmentName = (PropertiesService.getScriptProperties().getProperty('attachment') != null) ?
    PropertiesService.getScriptProperties().getProperty('attachment') :
    e.formInput.attachments
  let attachmentType
  let invoiceFiles = sheets[data.sheet].drive.file.folder
  let invoiceImage = sheets[data.sheet].drive.image.folder
  let key = ATL.getKey()
  let fileName = (key + '.file.' + ATL.getKey()) + '.pdf'
  let filePath
  if (attachmentName == 'Email body as attachment') {
    let temp = invoiceFiles.createFile("Invoice Temp" + ".txt", message.getBody(), "text/html");
    invoiceFiles.createFile(temp.getAs("application/pdf")).setName(fileName)
    attachmentType = 'file'
    filePath = invoiceFiles.getName() + "/" + fileName
    temp.setTrashed(true);
  }
  if (attachmentName == 'No attachment') {
  }

  message.getAttachments().forEach(attachment => {
    if (attachmentName == attachment.getName()) {
      if (attachment.getContentType() == 'application/pdf') {
        invoiceFiles.createFile(attachment.copyBlob()).setName(fileName)
        attachmentType = 'file'
        filePath = invoiceFiles.getName() + "/" + fileName
      } else {
        invoiceImage.createFile(attachment.getBlob()).setName(fileName)
        attachmentType = 'image'
        filePath = invoiceImage.getName() + "/" + fileName
      }
    }
  })
  return { key: key, type: attachmentType, filePath: filePath }
}
/**
 *  Click action
 *  @Param {Object} data - event object
 *  @Return {Object} 
 */
function splitButtonClicked(e) {
  console.log("splitButtonClicked(e) called")
  return processFormData(e)
}
/**
 *  Check form completeness
 *  @Param {Object} data - event object
 *  @Return {Card} gotoBillDataCard(e)|gotoSplitCard(e)
 */
function processFormData(e) {
  console.log("processFormData(e) called")

  if (hasAttachment(e)) {
    addAttachmentToScriptProperties(e)
  }
  if (!hasBillData(e)) {
    return gotoBillDataCard(e)
  }
  if (hasBillData(e) && hasSplitData(e)) {
    addSplitDataToScriptProperties(e)
    addBillDataToScriptProperties(e)
    return gotoSplitCard(e)
  }
  if (hasBillData(e)) {
    addBillDataToScriptProperties(e)
    return gotoSplitCard(e)
  }
}
/**
 *  Requires the following input feilds
 *  @param {Object} e - event object
 *  @param {String} e.formInput.lineitem
 *  @param {String} e.formInput.property
 *  @param {String} e.formInput.itemAmount
 *  @Return {Boolean} 
 */
function hasSplitData(e) {
  console.log("hasSplitData(e) called");
  return (e.formInput.lineitem != null && e.formInput.property != null && e.formInput.itemAmount != null)
}
/**
 *  @Param {Object} data - event object
 */
function addSplitDataToScriptProperties(e) {
  console.log("addSplitDataToScriptProperties(e) called")
  PropertiesService.getScriptProperties()
    .setProperty('lineitems', JSON.stringify([[e.formInput.lineitem,
    e.formInput.itemAmount,
    e.formInput.property]]
    )
    )
}

function hasAttachment(e) {
  console.log(' hasAttachment(e) called')
  return (e.formInput.attachments != null)
}
function addAttachmentToScriptProperties(e) {
  console.log("addAttachmentToScriptProperties(e) called")
  PropertiesService.getScriptProperties().setProperty('attachment', e.formInput.attachments)

}
/**
 *  Requires the following input feilds
 *  @param {Object} e - event object
 *  @param {String} e.formInput.vendor
 *  @param {String} e.formInput.refNum
 *  @param {String} e.formInput.dueDate
 *  @Return {Boolean} 
 */
function hasBillData(e) {
  console.log("checkForBillData(e) called");
  return (e.formInput.vendor != null && e.formInput.refNum != null && e.formInput.dueDate != null)
}
/**
 *  @Param {Object} data - event object
 */
function addBillDataToScriptProperties(e) {
  console.log("addBillDataToScriptProperties(e) called")
  PropertiesService.getScriptProperties()
    .setProperty('invoiceData', JSON.stringify({
      'Vendor': e.formInput.vendor,
      'Invoice Number': e.formInput.refNum,
      'Due Date': e.formInput.dueDate.msSinceEpoch
    }))
}
/**
 *  @Param {Object} data - event object
 *  @Return {Card}
 */
function gotoBillDataCard(e) {
  console.log("gotoBillDataCard(e) called");
  let navigation = CardService.newNavigation().pushCard(billDataCard(e))
  return CardService.newActionResponseBuilder().setNavigation(navigation).build();
}
/**
 *  @Param {Object} data - event object
 *  @Return {Card}
 */
function gotoSplitCard(e) {
  console.log("gotoSplitCard(e) called");
  let navigation = CardService.newNavigation().pushCard(splitCard(e))
  return CardService.newActionResponseBuilder().setNavigation(navigation).build();
}
