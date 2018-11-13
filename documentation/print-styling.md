# tl;dr

## Works with most

### Define Page and Set Margins

```css
  /* Name can be given as @page name {} */
  @page { 
     margin: 3cm;
  }

  /* Optional page distinction, :right is available as well */
  @page :left { 
     margin: 3cm;
  }

  /* Prevent break-through on elements */
  p a {
    word-wrap: break-word;
  }


```

Force a page break after an element - good for knocking a column down or if you have a few rows of elements that need printed - consider wrapping them in a block.  For example, if you have 12 rows, and only 4 can reliably fit on a page before potentially getting cut off in the middle, chunk the array (or enumerable) and wrap items inside of an element you can break after.

```css
div.item-set {
  /* Before may also be used */
  page-break-after: always;
}

```

## Works with some

Will not work with many browsers, but some will process this correct and attempt not to page break on this element.

```css
.item {
  page-break-inside: avoid;
}
```

# Resources
http://coding.smashingmagazine.com/2011/11/24/how-to-set-up-a-print-style-sheet/
