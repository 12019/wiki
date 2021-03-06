
```
<html>
<head>
<style>
div {
  border: 2px solid red;
  padding: 12px;
  line-height: 1.66666667;
  width: 70px;
}
</style>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
</head>
<body>
  <div contenteditable id="editor">
    The caret
    <span id="line2"> height should be the same for each line.</span>
  </div>
<script>
test(function () {
  var editor = document.getElementById('editor');
  editor.focus();

  var caretHeight1 = window.internals.absoluteCaretBounds().height;
  var sel = window.getSelection();
  sel.collapse(line2, 0);
  var caretHeight2 = window.internals.absoluteCaretBounds().height;

  assert_equals(caretHeight1, caretHeight2, 'The caret height is the same for each line.');
}, 'Compare the caret height');
</script>
</body>
</html>
```

# editing/pasteboard/restore-collapsed-space-for-copy.html
```
commit 131df048e7e445a03706078367285112e6b3467d
Author: joone.hur <joone.hur@intel.com>
Date:   Wed Sep 14 07:33:08 2016 -0700

    Restore a collapsed leading space of text used for line break
    
    When a text is wrapped during layout, the leading space of the
    text can be collapsed and a line break is inserted instead of
    the space. In this case, we need to restore the collapsed space
    when we copy the text.
    
    This CL handles the case that the below CL didn't cover:
    https://codereview.chromium.org/2320533002/
    
    In addition, while iterating through the DOM range, the below
    case with the TextIteratorBehavior flag don't need to restore
    the leading space.
    
    * Running DumpRenderTree : TextIteratorForInnerText
    * Searching text : TextIteratorDoesNotBreakAtReplacedElement
    * Getting a plain text for copy & paste: TextIteratorEmitsImageAltText
    
    BUG=318925
    TEST=editing/pasteboard/restore-collapsed-space-for-copy.html
    
    Review-Url: https://codereview.chromium.org/2325553002
    Cr-Commit-Position: refs/heads/master@{#418557}
```
```
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<script src="../assert_selection.js"></script>
<script>
test(() => assert_selection(
  '<div style="width: 10em;">Copy this area <a href="http://foo/">AVeryLongWordThatWillWrap</a></div><div contenteditable>|</div>',
  selection => {
    selection.setClipboardData('Copy this area <a href="http://foo/">AVeryLongWordThatWillWrap</a>');
    selection.document.execCommand('paste');
  },
  '<div style="width: 10em;">Copy this area <a href="http://foo/">AVeryLongWordThatWillWrap</a></div><div contenteditable>Copy this area <a href="http://foo/">AVeryLongWordThatWillWrap|</a></div>'),
  '1. Restore the collapsed trailing space');

test(() => assert_selection(
  '<div style="width: 2em;"><b><i>foo </i></b>bar</div><div contenteditable>|</div>',
  selection => {
    selection.setClipboardData('<b><i>foo </i></b>bar');
    selection.document.execCommand('paste');
  },
 '<div style="width: 2em;"><b><i>foo </i></b>bar</div><div contenteditable><b><i>foo </i></b>bar|</div>'),
  '2. Restore the collapsed trailing space');

test(() => assert_selection(
  '<div style="width: 2em;"><b><i>foo</i></b> bar</div><div contenteditable>|</div>',
  selection => {
    selection.setClipboardData('<b><i>foo</i></b> bar');
    selection.document.execCommand('paste');
  },
  '<div style="width: 2em;"><b><i>foo</i></b> bar</div><div contenteditable><b><i>foo</i></b> bar|</div>'),
  '3. Restore the collapsed leading space');

test(() => assert_selection(
  '<div style="width: 2em;">작은홍띠점박이푸른부전나비</div><div contenteditable>|</div>',
  selection => {
    selection.setClipboardData('작은홍띠점박이푸른부전나비');
    selection.document.execCommand('paste');
  },
 '<div style="width: 2em;">작은홍띠점박이푸른부전나비</div><div contenteditable>작은홍띠점박이푸른부전나비|</div>'),
  '4. Space should not be added for CJK');

test(() => assert_selection(
  '<div style="width: 2em; word-break: break-all">Pneumonoultramicroscopicsilicovolcanoconiosis</div><div contenteditable>|</div>',
  selection => {
    selection.setClipboardData('Pneumonoultramicroscopicsilicovolcanoconiosis');
    selection.document.execCommand('paste');
  },
 '<div style="width: 2em; word-break: break-all">Pneumonoultramicroscopicsilicovolcanoconiosis</div><div contenteditable>Pneumonoultramicroscopicsilicovolcanoconiosis|</div>'),
  '5. Space should not be added for CSS word-break: break-all');
</script>
```





# editing/inserting/insert-space.html
```
commit f9400ff295d15df932a5ffc903e4ab42163b1b4d
Author: joone.hur <joone.hur@intel.com>
Date:   Sun Oct 30 19:31:54 2016 -0700

    Add a test case for bug 659109
    
    <div contenteditable="true"><br> </div>
    
    When we run the above example with M54/55, mutiple
    spaces can't be added under <div> because two plain
    spaces are collapsed into one space while typing.
    
    This is a regression caused by https://codereview.chromium.org/2175163004.
    It was fixed in M56: https://codereview.chromium.org/2432083003.
    
    This test case is added to reproduce the problem mentioned in the bug report.
    
    BUG=659109
    TEST=editing/inserting/insert-space.html
    
    Review-Url: https://codereview.chromium.org/2451423003
    Cr-Commit-Position: refs/heads/master@{#428639}
```

```
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<script src="../assert_selection.js"></script>
<script>
test(() => assert_selection(
  '<div contenteditable><p>A|B</p></div>',
  'insertText \ ',
  '<div contenteditable><p>A |B</p></div>'),
  'insert a plain space in the middle of text node');

test(() => assert_selection(
  '<div contenteditable><p id="para"></p></div>',
  selection => {
    var para = selection.document.getElementById('para');
    para.appendChild(selection.document.createTextNode('A'));
    para.appendChild(selection.document.createTextNode('B'));
    selection.collapse(para.firstChild, 1);

    selection.document.execCommand('insertText', false, ' ');
  },
  '<div contenteditable><p id="para">A |B</p></div>'),
  'insert a plain space between two inserted text nodes');

test(() => assert_selection(
  '<div contenteditable><p id="para"></p></div>',
  selection => {
    var para = selection.document.getElementById('para');
    para.appendChild(selection.document.createTextNode('A'));
    para.appendChild(selection.document.createTextNode(''));
    selection.collapse(para.firstChild, 1);

    selection.document.execCommand('insertText', false, ' ');
  },
  '<div contenteditable><p id="para">A\u00A0|</p></div>'),
  'Insert a &nbsp; instead of plain space when it is inserted before the empty text node');

test(() => assert_selection(
  '<div contenteditable><p id="para"></p></div>',
  selection => {
    var para = selection.document.getElementById('para');
    para.appendChild(selection.document.createTextNode('A'));
    para.appendChild(selection.document.createTextNode(' B'));
    selection.collapse(para.firstChild, 1);

    selection.document.execCommand('insertText', false, ' ');
  },
  '<div contenteditable><p id="para">A\u00A0| B</p></div>'),
  'Insert a &nbsp; instead of plain space when it is inserted before the text node that has a leading plain space');

test(() => assert_selection(
  '<div contenteditable>|<br> </div>',
  selection => {
    selection.document.execCommand('insertText', false, ' ');
    selection.document.execCommand('insertText', false, ' ');
    selection.document.execCommand('insertText', false, ' ');
  },
  '<div contenteditable>\u00A0 \u00A0| </div>'),
  'Insert spaces into the editable <div> that only has <br> and space as child');
</script>
```


# editing/pasteboard/cut-paste-formatting-tag.html

```
commit f77d0eaedb2787483c147b973bcf7e7de82bf0f3
Author: joone.hur <joone.hur@intel.com>
Date:   Fri Aug 12 23:47:05 2016 -0700

    Keep formatting tags included when it is cut or copied.

    When we copy/cut a formatting tag without the highest node.
    the formatting tag's text can be wrapped by <span> tag
    instead of the formatting tag. This CL allows to keep
    formatting tags included when it is cut or copied.

    BUG=634482
    TEST=editing/pasteboard/cut-paste-formatting-tag.html

    Review-Url: https://codereview.chromium.org/2229703004
    Cr-Commit-Position: refs/heads/master@{#411883}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<script src="../assert_selection.js"></script>
<script>

test(() => assert_selection(
  '<div contenteditable style="width: 300px; height: 250px; border: 1px solid black"><div><b>^<br></b></div><b>foo|</b></div>',
  selection => {
    selection.document.execCommand('cut');
    selection.document.execCommand('paste');
  },
  '<div contenteditable style="width: 300px; height: 250px; border: 1px solid black"><br><div><b>foo|</b></div></div>'),
  'Cut all tags and paste them');

</script>

```
## editing/inserting/insert-space.html
```
commit 4dab74137593abb0888e415294aeb80da27362e3
Author: joone.hur <joone.hur@intel.com>
Date:   Thu Jul 28 20:35:48 2016 -0700

    Add &nbsp; instead of plain space when it is inserted before the empty text node
    
    This CL fixes the regression that space key doesn't work in the gmail
    chat window.
    
    BUG=632300
    TEST=editing/inserting/insert-space.html
    
    Review-Url: https://codereview.chromium.org/2191163002
    Cr-Commit-Position: refs/heads/master@{#408571}

commit 8134eeb454ef76c72df8cd7c26f6141072c314cb
Author: joone.hur <joone.hur@intel.com>
Date:   Tue Jul 26 17:04:40 2016 -0700

    Add a plain space instead of &nbsp; between text nodes
    
    When we rebalance white spaces, &nbsp; can be added as space
    under some conditions. This CL adds a condition that the next
    sibling text node should not exist.
    
    BUG=310149
    TEST=editing/inserting/insert-space.html
    
    Review-Url: https://codereview.chromium.org/2175163004
    Cr-Commit-Position: refs/heads/master@{#407981}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<script src="../assert_selection.js"></script>
<script>
test(() => assert_selection(
  '<div contenteditable><p>A|B</p></div>',
  'insertText \ ',
  '<div contenteditable><p>A |B</p></div>'),
  'insert a plain space in the middle of text node');

test(() => assert_selection(
  '<div contenteditable><p id="para"></p></div>',
  selection => {
    var para = selection.document.getElementById('para');
    para.appendChild(selection.document.createTextNode('A'));
    para.appendChild(selection.document.createTextNode('B'));
    selection.collapse(para.firstChild, 1);

    selection.document.execCommand('insertText', false, ' ');
  },
  '<div contenteditable><p id="para">A |B</p></div>'),
  'insert a plain space between two inserted text nodes');

test(() => assert_selection(
  '<div contenteditable><p id="para"></p></div>',
  selection => {
   var para = selection.document.getElementById('para');
    para.appendChild(selection.document.createTextNode('A'));
    para.appendChild(selection.document.createTextNode(''));
    selection.collapse(para.firstChild, 1);

    selection.document.execCommand('insertText', false, ' ');
  },
  '<div contenteditable><p id="para">A\u00A0|</p></div>'),
  'Insert a &nbsp; instead of plain space when it is inserted before the empty text node');
</script>
```

## editing/execCommand/indent/indent_empty_quote.html
```
commit b844ddd6c6b82784257c183e50244eff895bce09
Author: joone.hur <joone.hur@intel.com>
Date:   Fri Jul 22 00:14:42 2016 -0700

    Remove the unnecessary blockquote after indenting an empty blockquote
    
    Here is a DOM state when we only add a blockquote.
    <div contenteditable><blockquote>|<br></blockquote></div>
    
    After indenting the blockquote, the change is as follows:
    <div contenteditable>
      <blockquote>
        <blockquote style="margin: 0 0 0 40px; border: none; padding: 0px;">
          <blockquote>|<br></blockquote>
        </blockquote>
      </blockquote>
    </div>
    
    There is an unnecessary blockquote so the result should be as follows:
    
    <div contenteditable>
      <blockquote style="margin: 0 0 0 40px; border: none; padding: 0px;">
        <blockquote>|<br></blockquote>
      </blockquote>
    </div>
    
    This CL removes the additional blockquote.
    
    BUG=625802
    TEST=editing/execCommand/indent-empty-quote.html
    
    Review-Url: https://codereview.chromium.org/2175433002
    Cr-Commit-Position: refs/heads/master@{#407091}
```
```javascript
<!doctype html>
<script src="../../../resources/testharness.js"></script>
<script src="../../../resources/testharnessreport.js"></script>
<script src="../../assert_selection.js"></script>
<div id="log"></div>
<script>
test(() => {
  assert_selection(
    '<div contenteditable><blockquote>|<br></blockquote></div>',
    'indent',
    '<div contenteditable><blockquote style="margin: 0 0 0 40px; border: none; padding: 0px;"><blockquote>|<br></blockquote></blockquote></div>');
}, 'indent empty <blockquote>');
test(() => {
  assert_selection(
    '<div contenteditable><blockquote>1|</blockquote></div>',
    'indent',
    '<div contenteditable><blockquote style="margin: 0 0 0 40px; border: none; padding: 0px;"><blockquote>1|</blockquote></blockquote></div>');
}, 'indent <blockquote> with text');
</script>
```


## editing/assert_selection.html
```
commit 674e1ec92f839b188bf14498ee6c6bd9489e330a
Author: joone.hur <joone.hur@intel.com>
Date:   Tue Jul 19 00:15:10 2016 -0700

    The parameter of the tester should have spaces in assert_selection()
    
    When we run assert_selection(), the tester can take one parameter, but
    it does not allow to have spaces. As a result, it cannot handle HTML
    tags as parameter. This CL allows the parameter to have spaces.
    
    BUG=none
    TEST=editing/assert_selection.html
    
    Review-Url: https://codereview.chromium.org/2155793002
    Cr-Commit-Position: refs/heads/master@{#406226}
```
```javascript
test(() => {
    assert_selection(
        '<div contenteditable><p>^test|</p></div>',
        'insertHTML <span style="color: green">green</span>',
        '<div contenteditable><p><span style="color: green">green|</span></p></div>');
}, 'multiple spaces in function');

```

## editing/execCommand/crash-inserting-span.html
```
commit e7f2bcf1c9f2c9f7d998328cc47face9b75cca1e
Author: joone.hur <joone.hur@intel.com>
Date:   Fri Jul 15 02:43:59 2016 -0700

    Add null check to fix crash in blink::ComputedStyle::display
    
    These is a case that node->computedStyle() returns null
    so it needs to check whether computedStyle() returns
    null or not.
    
    BUG=626991
    TEST=editing/execCommand/crash-inserting-span.html
    
    Review-Url: https://codereview.chromium.org/2135993003
    Cr-Commit-Position: refs/heads/master@{#405733}
```
```javascript
<!doctype html>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<script src="../assert_selection.js"></script>
<div id="log"></div>
<script>
test(() => {
  assert_selection(
    '<div contenteditable><span><a href="foo"><p>^test</p><p>foo<span>bar</span>foo<span>bar</span>.|</p></a></span></div>',
    'insertHTML <span>green</span>',
    '<div contenteditable><span><p><a href="foo">green|</a></p></span></div>');
}, 'Replace the selected tags with span tags');
</script>
```
## editing/pasteboard/copy-paste-white-space.html
```
commit 3564239b151f615afca39cfd886d225a2b4740f8
Author: joone.hur <joone.hur@intel.com>
Date:   Sun Jul 3 23:39:16 2016 -0700

    Aware of pre-line value of CSS white-space property when copying a selection
    
    When we copy a selection that has white-space(pre-line) style,
    newlines are not preserved.
    
    This CL allows to preserve newlines when elements has white-space(pre-line) style.
    
    BUG=317365
    TEST=editing/pasteboard/copy-paste-white-space.html
    
    Review-Url: https://codereview.chromium.org/2119763002
    Cr-Commit-Position: refs/heads/master@{#403644}
```
```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<div id="text">
    Testing 
CSS white-space property
</div>
<textarea id="textarea" rows="4" cols="40"></textarea>
<script>
  var textForCopy = document.getElementById('text');
  var textarea = document.getElementById('textarea');

function defocus() {
  eventSender.mouseMoveTo(0, 0);
  eventSender.mouseDown();
  eventSender.mouseUp();
}

function copyAndPaste(whiteSpaceValue) {
  defocus();
  textarea.value ='';
  textForCopy.style.whiteSpace = whiteSpaceValue;
  var selection = window.getSelection();
  selection.removeAllRanges();
  var range = document.createRange();
  range.selectNode(textForCopy);
  selection.addRange(range);

  document.execCommand('copy');
  textarea.focus();
  document.execCommand('paste');

  return whiteSpaceValue;
}

test(function() {
  assert_equals(textarea.value, '\n  \tTesting \nCSS white-space property\n');
}, copyAndPaste('pre'));

test(function() {
  assert_equals(textarea.value, '\n  \tTesting \nCSS white-space property\n');
}, copyAndPaste('pre-wrap'));

test(function() {
  assert_equals(textarea.value, 'Testing CSS white-space property\n');
}, copyAndPaste('normal'));
test(function() {
  assert_equals(textarea.value, 'Testing CSS white-space property\n');
}, copyAndPaste('nowrap'));

test(function() {
  assert_equals(textarea.value, '\nTesting \nCSS white-space property\n');
}, copyAndPaste('pre-line'));
</script>
```



## editing/deleting/backspace-merge-into-block.html

```
commit 5220699381aae450f22d1ea3805141deb139f8f3
Author: joone.hur <joone.hur@intel.com>
Date:   Tue Jun 28 19:15:00 2016 -0700

    Remove style spans to follow the styles of the block element
    
    This CL removes style spans to follow the styles of the block
    element(li, pre, td, and h1~6) when the text of the pasted
    or merged element becomes a part of the block element.
    
    BUG=226941
    TEST=third_party/WebKit/LayoutTests/editing/deleting/backspace-merge-into-block-element.html
    
    Review-Url: https://codereview.chromium.org/2102913002
    Cr-Commit-Position: refs/heads/master@{#402659}
```

```javascript
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<script src="../assert_selection.js"></script>
<style>
p {
    font-size: 20px;
    line-height: 22px;
    color: red;
}
</style>
<div id="log"></div>
<script>
test(() => {
    assert_selection(
        '<div contenteditable="true"><h1>Heading 1:</h1>^<p>|paragraph was merged.</p></div>',
        'delete',
        '<div contenteditable="true"><h1>Heading 1:|paragraph was merged.</h1></div>',
        'Make a paragraph into a heading');
    assert_selection(
        '<div contenteditable="true"><pre>Preformatted text:</pre>^<p>|paragraph was merged.</p></div>',
        'delete',
        '<div contenteditable="true"><pre>Preformatted text:|paragraph was merged.</pre></div>',
        'Make a paragraph into a pre');
    assert_selection(
        '<div contenteditable="true"><ul><li>List Item:</li></ul>^<p>|paragraph was merged.</p></div>',
        'delete',
        '<div contenteditable="true"><ul><li>List Item:|paragraph was merged.</li></ul></div>',
        'Make a paragraph into a list');
    assert_selection(
        '<div contenteditable="true"><table><tbody><tr><td>Table:</td></tr></tbody></table>^<p>|paragraph was merged.</p></div>',
        'delete',
        '<div contenteditable="true"><table><tbody><tr><td>Table:|paragraph was merged.</td></tr></tbody></table></div>',
        'Make a paragraph into a table');
}, 'merge into a block by backspace');
</script>
```



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
