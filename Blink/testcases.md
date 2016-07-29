## editing/deleting/backspace-merge-into-list-item.html
```
commit bb59e5b729913a2023ea70e50d0689626565dd7e
Author: joone.hur <joone.hur@intel.com>
Date:   Wed Jun 22 20:56:12 2016 -0700

    Remove style spans to follow the styles of list item
    
    This CL removes style spans to follow the styles of list item when
    the text of the pasted or merged element becomes a list item.
    
    BUG=335955
    TEST=editing/deleting/backspace-merge-into-list-item.html
    
    Review-Url: https://codereview.chromium.org/2072093002
    Cr-Commit-Position: refs/heads/master@{#401533}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<style>
#editable p { 
  font-size: 20px;
  line-height: 22px;
  color: red;
}
</style>
<div contenteditable="true" id="editable">
  <ul>
    <li>list item 1</li>
    <li>list item 2</li>
    <li>list item 3</li>
  </ul>
  <p>Paragraph</p>
</div>
<script>
test(function() {
  var editor = document.getElementById('editable');
  var range = document.createRange();
  var selection = window.getSelection();
  range.setStart(editor.childNodes[2], 0);
  range.collapse(true);
  selection.removeAllRanges();
  selection.addRange(range);
  editor.focus();
  document.execCommand('delete');

  var htmlPara = document.getElementsByTagName('li')[2].outerHTML;
  assert_equals(htmlPara, '<li>list item 3Paragraph</li>');
}, 'make a paragraph into a list by backspace');
</script>
```

## editing/deleting/backspace-merge-two-paragraphs.html
```
commit 8e411d16171d27612776a2f05356b0ed9f06b848
Author: joone.hur <joone.hur@intel.com>
Date:   Sun Jun 12 23:55:43 2016 -0700

    Don't need to preserve CSS line-height property during editing operation
    
    When we merge two paragraphs by typing backspace key at the head
    of the second paragraph, the styles of the second paragraph can be
    preserved by using <span> tag. However, we don't need to preserve
    line-height style because the computed value can be different from
    the value defined in HTML.
    
    BUG=226941
    TEST=editing/deleting/backspace-merge-two-paragraphs.html
    
    Review-Url: https://codereview.chromium.org/2064473002
    Cr-Commit-Position: refs/heads/master@{#399410}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<style>
div {
  border: 1px solid gray;
  padding: 10px;
  line-height: 1.44;
}
</style>
<div contenteditable="true" id="editable">
  <p>This is the first paragraph.</p>
  <p>This is the second.</p>
</div>
<script>
test(function() {
  var editor = document.getElementById('editable');
  var range = document.createRange();
  var selection = window.getSelection();
  range.setStart(editor.childNodes[2], 0);
  range.collapse(true);
  selection.removeAllRanges();
  selection.addRange(range);
  editor.focus();
  document.execCommand('delete', null, false);

  var html_para = document.getElementsByTagName('p')[0].outerHTML;
  assert_equals(html_para, '<p>This is the first paragraph.This is the second.</p>');
}, 'merge two paragraphs by backspace');
</script>
```

## editing/execCommand/queryCommandState-04.html
```
commit dfe3d34820670521568ca7dfe681f701da94325b
Author: joone.hur <joone.hur@intel.com>
Date:   Mon May 30 17:48:24 2016 -0700

    Apply vertical-align style of <sub> and <sup> to child elements.
    
    If the selected element has <sub> or <sup> ancestor element,
    apply the corresponding style(vertical-align) to it so that
    document.queryCommandState() works fine on the selected element.
    
    <sub> and <sup> tags are represented with CSS vertical-align
    property but they are not inherited. Therefore, we need to apply the
    property to child elements because <sub> and <sup> tag can have
    child elements such as <i> or <b> tag.
    
    BUG=582225
    TEST=editing/execCommand/queryCommandState-04.html
    
    Review-Url: https://codereview.chromium.org/1986563002
    Cr-Commit-Position: refs/heads/master@{#396763}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<div contenteditable="true">
<sub><i>1. Make text subscript and italic</i></sub>
<sup><i>2. Make text superscript and italic</i></sup>
</div>
<div id="log"></div>
<script>
function test_sub() {
    document.getSelection().removeAllRanges();
    var range = document.createRange();
    range.selectNode(document.querySelector('i'));
    document.getSelection().addRange(range);

    assert_equals(document.queryCommandState('subscript'), true);
}

function test_sup() {
    document.getSelection().removeAllRanges();
    var range = document.createRange();
    range.selectNode(document.querySelectorAll('i')[1]);
    document.getSelection().addRange(range);

    assert_equals(document.queryCommandState('superscript'), true);
}

function test_all(platform) {
    if (platform)
        internals.settings.setEditingBehavior(platform);
    test(test_sub, `${platform}: run queryCommandState('subscript')`);
    test(test_sup, `${platform}: run queryCommandState('superscript')`);
}

(window.internals ? ['mac', 'win',''] : ['']).forEach(test_all);
</script>
```

## editing/execCommand/queryCommandState-03.html
```
commit 1e7d66ca32345881ae135af6ed712d3571f9b9e9
Author: joone.hur <joone.hur@intel.com>
Date:   Tue May 10 21:54:09 2016 -0700

    Do not ignore the text style properties when checking styles in EditingStyle.
    
    When we run document.queryCommandState() to see if the text styles(bold,
    italic, underline, strikethrough) were applied on selected img tag, it
    returns wrong value because the text styles are only considerded for text node.
    
    BUG=584939
    TEST=editing/execCommand/queryCommandState-03.html
    
    Review-Url: https://codereview.chromium.org/1960553002
    Cr-Commit-Position: refs/heads/master@{#392855}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<div contenteditable="true">
<img src="../resources/abe.png">
</div>
<script>
test(function() {
    document.getSelection().removeAllRanges();
    var range = document.createRange();
    range.selectNode(document.querySelector('img'));
    document.getSelection().addRange(range);

    assert_equals(document.queryCommandState('bold'), false);
    assert_equals(document.queryCommandState('italic'), false);
    assert_equals(document.queryCommandState('underline'), false);
    assert_equals(document.queryCommandState('strikethrough'), false);
}, 'run queryCommandState() on the selected img element');
</script>
```
