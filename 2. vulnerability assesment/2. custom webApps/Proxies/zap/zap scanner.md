# ZAP Scanner

---

ZAP also comes bundled with a Web Scanner similar to Burp Scanner. ZAP Scanner is capable of building site maps using ZAP Spider and performing both passive and active scans to look for various types of vulnerabilities.

we can check the Sites tab on the main ZAP UI, or we can click on the first button on the right pane (`Sites Tree`), which should show us an expandable tree-list view of all identified websites and their sub-directories:

# ajax spider
`Ajax Spider`, which can be started from the third button on the right pane. The difference between this and the normal scanner is that Ajax Spider also tries to identify links requested through JavaScript AJAX requests, which may be running on the page even after it loads. Try running it after the normal Spider finishes its scan, as this may give a better output and add a few links the normal Spider may have missed, though it may take a little bit longer to finish.