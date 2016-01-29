# Link CC and BCC
Created by Aymeric Hermant - 2016-01-28

This is an update of the Link plugin of CKeditor in order to add some CC and BCC fields to the mailtos by clicking on the link icon and selecting the Email option.
It contains a link.js file and some english(canadian) and french language files for CKEditor 4.

##Installation
To install it just copy and past the files to your CKeditor folder :
* link.js to the \plugins\link\dialogs\ folder of CKeditor
* the language files to the \lang folder of CKeditor

##Troubleshots
This patch may not be compliant with the version of ckeditor over 2016-01-28. If it is the case you can update the *link* plugin by yourself by following those steps :

### Read the link.js file
* Get a text editor with a prettifier. For instance, Sublime text with the prettify package.
* In you ckeditor folder, open the file \plugins\link\dialogs\link.js and prettify it

### Add the fields to the Email link editor
* Find the inputs creation related to the Emails. They come with some setup: function(F) and commit: function(F). Add your own CC & CCI fields
````javascript
               type: 'vbox',
                id: 'emailOptions',
                padding: 1,
                children: [{
                    type: 'text',
                    id: 'emailAddress',
                    label: E.emailAddress,
                    required: true,
                    validate: function() {
                        var F = this.getDialog();
                        if (!F.getContentElement('info', 'linkType') || F.getValueOf('info', 'linkType') != 'email') return true;
                        var G = CKEDITOR.dialog.validate.notEmpty(E.noEmail);
                        return G.apply(this);
                    },
                    setup: function(F) {
                        if (F.email) this.setValue(F.email.address);
                        var G = this.getDialog().getContentElement('info', 'linkType');
                        if (G && G.getValue() == 'email') this.select();
                    },
                    commit: function(F) {
                        if (!F.email) F.email = {};
                        F.email.address = this.getValue();
                    }
                },  {
                    // Email CC field here
                    type: 'text',
                    id: 'emailCC',
                    label: E.emailCC,
                    setup: function(F) {
                        if (F.email) this.setValue(F.email.CC);
                    },
                    commit: function(F) {
                        if (!F.email) F.email = {};
                        F.email.CC = this.getValue();
                    }
                },  {
                    // email CCI field here
                    type: 'text',
                    id: 'emailCCI',
                    label: E.emailCCI,
                    setup: function(F) {
                        if (F.email) this.setValue(F.email.CCI);
                    },
                    commit: function(F) {
                        if (!F.email) F.email = {};
                        F.email.CCI = this.getValue();
                    }
                },{
                    type: 'text',
                    id: 'emailSubject',
                    label: E.emailSubject,
                    setup: function(F) {
                        if (F.email) this.setValue(F.email.subject);
                    },
                    commit: function(F) {
                        if (!F.email) F.email = {};
                        F.email.subject = this.getValue();
                    }
                },
```
* Find the case 'encode' to encode your email address components in HTML and build the URL from the editor 
````javascript
                        case 'encode':
                            // encoding of the fields in HTML (including CC and BCC)
                            var R = encodeURIComponent(P.subject || ''),
                                S = encodeURIComponent(P.body || ''),
                                U = encodeURIComponent(P.CC || ''),
                                V = encodeURIComponent(P.CCI || ''),
                                T = [];
                            U && T.push('cc=' + U);
                            V && T.push('bcc=' + V);
                            R && T.push('subject=' + R);
                            S && T.push('body=' + S);                      
                            T = T.length ? '?' + T.join('&') : '';
                            if (y == 'encode') {
                                O = ["javascript:void(location.href='mailto:'+", B(Q)];
                                T && O.push("+'", x(T), "'");
                                O.push(')');
                            } else O = ['mailto:', Q, T];
                            break;
```

### Edit your mailto link and get the values in the right fields
* Find the list of regexp in the link.js file and add one regexp for the CC field and one another for the BCC. They should be identical to the subject regexp. 
```javascript
        //Regexp to identify the cc and bcc in the mailto string (fa for cc, fb for bcc)
        e = /^javascript:/,
        f = /^mailto:([^?]+)(?:\?(.+))?$/,
        // beginning of the CC, BCC regexp
        fa = /cc=([^;?:@&=$,\/]*)/,
        fb = /bcc=([^;?:@&=$,\/]*)/,
        // end of the CC, BCC regexp
        g = /subject=([^;?:@&=$,\/]*)/,
        h = /body=([^;?:@&=$,\/]*)/,
        i = /^#(.*)$/,
        j = /^((?:http|https|ftp|news):\/\/)?(.*)$/,
        k = /^(_(?:self|top|parent|blank))$/,
        l = /^javascript:void\(location\.href='mailto:'\+String\.fromCharCode\(([^)]+)\)(?:\+'(.*)')?\)$/,
        m = /^javascript:([^(]+)\(([^)]+)\)$/,
        n = /\s*window.open\(\s*this\.href\s*,\s*(?:'([^']*)'|null)\s*,\s*'([^']*)'\s*\)\s*;\s*return\s*false;*\s*/,
        o = /(?:^|,)([^=]+)=(\d+|yes|no)/gi,
```
* Go to the part of the code where the author identify the type of link (basically search for the match methods or the email string) and apply your regexp by using a match method against the href string (H)
```javascript
            if (!M.type)
                if (K = H.match(i)) {
                    M.type = 'anchor';
                    M.anchor = {};
                    M.anchor.name = M.anchor.id = K[1];
                } else if (J = H.match(f)) {
                // Check of the regexp Vs the html content (U=CC, V=BCC, N=Subject, O=Body)    
                var U = H.match(fa),
                    V = H.match(fb),
                    N = H.match(g),
                    O = H.match(h); 
                // End of the checking
                M.type = 'email';
                var P = M.email = {};
                P.address = J[1];
```
* A few lines below, decode the url components to remove the HTML chars
```javascript
                var P = M.email = {};
                P.address = J[1];
                //Decoding of the content
                U && (P.CC = decodeURIComponent(U[1]));
                V && (P.CCI = decodeURIComponent(V[1]));
                N && (P.subject = decodeURIComponent(N[1]));
                //End of decoding
                console.log(N); 
                O && (P.body = decodeURIComponent(O[1]));
```
* Here you go. You should now be able to see the different components of your mailto link in the editor

### Read the language files
* Get a text editor with a prettifier. For instance, Sublime text with the prettify package.
* In you ckeditor folder, open the file you need in \lang and prettify it

### Update the langage file
* At this stage you have some fields with no label which is interseting but not pretty. To improve it go to ckeditor/lang and edit the language files you need. You have to add the lines including emailCC and emailBCC to the *link* subclass of the language files you want to use
```javascript
    link: {
        toolbar: 'Link',
        other: '<other>',
        menu: 'Edit Link',
        title: 'Link',
        info: 'Link Info',
        target: 'Target',
        upload: 'Upload',
        advanced: 'Advanced',
        type: 'Link Type',
        toUrl: 'URL',
        toAnchor: 'Link to anchor in the text',
        toEmail: 'E-mail',
        targetFrame: '<frame>',
        targetPopup: '<popup window>',
        targetFrameName: 'Target Frame Name',
        targetPopupName: 'Popup Window Name',
        popupFeatures: 'Popup Window Features',
        popupResizable: 'Resizable',
        popupStatusBar: 'Status Bar',
        popupLocationBar: 'Location Bar',
        popupToolbar: 'Toolbar',
        popupMenuBar: 'Menu Bar',
        popupFullScreen: 'Full Screen (IE)',
        popupScrollBars: 'Scroll Bars',
        popupDependent: 'Dependent (Netscape)',
        popupLeft: 'Left Position',
        popupTop: 'Top Position',
        id: 'Id',
        langDir: 'Language Direction',
        langDirLTR: 'Left to Right (LTR)',
        langDirRTL: 'Right to Left (RTL)',
        acccessKey: 'Access Key',
        name: 'Name',
        langCode: 'Language Code',
        tabIndex: 'Tab Index',
        advisoryTitle: 'Advisory Title',
        advisoryContentType: 'Advisory Content Type',
        cssClasses: 'Stylesheet Classes',
        charset: 'Linked Resource Charset',
        styles: 'Style',
        rel: 'Relationship',
        selectAnchor: 'Select an Anchor',
        anchorName: 'By Anchor Name',
        anchorId: 'By Element Id',
        emailAddress: 'E-Mail Address',
        emailCC: 'Carbon Copy addresses',
        emailCCI: 'Blind Carbon Copy addresses',        
        emailSubject: 'Message Subject',
        emailBody: 'Message Body',
        noAnchors: '(No anchors available in the document)',
        noUrl: 'Please type the link URL',
        noEmail: 'Please type the e-mail address'
    },
```
* That's all done. You can enjoy to use CK !
