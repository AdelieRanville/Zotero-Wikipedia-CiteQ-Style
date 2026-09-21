# Wikipedia Cite Q – CSL style for Zotero

A [Citation Style Language](https://citationstyles.org/) (CSL) style for [Zotero](https://www.zotero.org/) that outputs references for Wikipedia using the [`{{Cite Q}}`](https://en.wikipedia.org/wiki/Template:Cite_Q) template. The template only needs the Wikidata item identifier (QID) of the source.

## Usage

1. **Install the style**: download `wikipedia-cite-q.csl`, then in Zotero go to *Settings → Cite → Styles → `+`* and select the file (or simply double-click the file).
2. **Store the QID** in the item's **Extra** field, on its own line, in this form:
   ```
   QID: Q42
   ```
   The line must be at the **top** of the Extra field (first or second line): the citation processor stops reading `Key: value` lines at the first line that is not of that form. Put other content (e.g. `PMID: ...`) below or in between only if it also follows the `Key: value` form.
   You can use Cita to get the QID : https://github.com/zotero-cita/zotero-cita

4. To generate a single citation : Select your items, right-click → *Create Bibliography from Items…* (or *Copy Citation / Bibliography*), choose **Wikipedia Cite Q**, and set the output mode to *Copy to Clipboard* (or use it from the word-processor plugin). 
5. Paste the result into your Wikipedia edit window.

## Output

The reference is wrapped in a named `<ref>` tag. The name follows an author-date pattern.

For an item with author "Smith", year 2020 and `QID: Q42` in **Extra**:

```
<ref name="Smith2020">{{Cite Q|Q42}}</ref>
```

For an item **without** a QID, a visible marker is output so the gap is easy to spot in your wikitext:

```
<ref name="Smith2020">{{Cite Q|QID MISSING}}</ref>
```

### Page numbers

If you enter a page in the Zotero citation dialog (the locator, with the label "Page"), a `page` parameter is added:

```
<ref name="Lamba2019">{{Cite Q|Q15625490|page=42}}</ref>
```

Only locators with the **page** label are used; other labels (chapter, section, ...) are ignored. Page numbers are only available in citations, not in the bibliography. The value is copied as typed (`42`, `42-45`, ...).

Note: on Wikipedia, two `<ref>` tags with the same name but different content (e.g. the same source cited at pages 42 and 50) produce a reference error. In that case, give one of the refs a different name by hand, or use a separate citation for each page.

### Ref name rules

| Item | Ref name |
|---|---|
| 1 author, 2020 | `Smith2020` |
| 2 authors, 2020 | `SmithJones2020` |
| 3+ authors, 2020 | first author + `EtAl` + year (see note below) |
| Same author and year, several items | `Smith2020a`, `Smith2020b` (year-suffix) |
| No author, but an editor | editor family name + year |
| No date | `Smith` + `nd` (`Smithnd`) |
| No author and no editor | the QID itself, e.g. `Q42` (no year) |
| Nothing available | no `name` attribute is output |

The name is always quoted (`name="..."`), so family names with spaces or particles stay valid.

The same output is used for citations and for the bibliography.



## How it works

Zotero passes the Extra field to the citation processor (citeproc-js), which reads `Key: value` lines at the top of the field as extra CSL variables (the "cheater syntax"). The key is kept **exactly as typed**, so `QID: Q42` becomes the variable `QID` (uppercase) and the style reads it with `<text variable="QID"/>`. A lowercase `qid: Q42` line is also accepted.

The `cite-q` macro builds the tag in four parts:

```xml
<text value="&lt;ref"/>
<text macro="ref-name-attribute"/>   <!-- name="Smith2020", omitted if empty -->
<text value="&gt;"/>
<!-- {{Cite Q|<qid>}} or {{Cite Q|QID MISSING}}, with |page=<locator> added by the page-param macro -->
<text value="&lt;/ref&gt;"/>
```

The `ref-name` macro uses the author (falling back to the editor, then the QID) in short form, plus the year and `year-suffix`. `disambiguate-add-year-suffix="true"` on `<citation>` adds the `a`, `b`, ... suffixes. The `<`, `>` and `"` characters are XML-escaped in the file.

## Customising

- **Without `<ref>` tags or name**: remove the `&lt;ref`, `ref-name-attribute`, `&gt;` and `&lt;/ref&gt;` parts of the `cite-q` macro to output only `{{Cite Q|Q42}}`.
- **Ref name pattern**: edit the `ref-name` macro (e.g. change the `et-al-min` value or the `et-al` term override in the `<locale>` block).
- **Missing-QID marker**: edit the text inside the `<else>` branch of the `cite-q` macro.
- **Author / ID**: change the `<author>` and `<id>` elements in the `<info>` block before publishing your own fork.

## Notes

- Multiple items in one citation are concatenated without a separator, which is what Wikipedia expects for adjacent `<ref>` tags.
- The QID is the only content of the template; author and date are read from the Zotero item only to build the ref name.
- With 3 or more authors, the processor may insert a space before `EtAl` (`"Smith EtAl2020"`). That is still a valid quoted ref name; adjust the `ref-name` macro if you prefer otherwise.
- `QID` is not a variable defined by the CSL specification, so the style will not pass schema validation for submission to the official CSL styles repository. It works in Zotero because citeproc-js accepts arbitrary variables from Extra.
- See the [CSL style development guide](https://github.com/citation-style-language/styles/blob/master/STYLE_DEVELOPMENT.md) for the CSL conventions followed here.

## License

Style released under [CC BY-SA 3.0](http://creativecommons.org/licenses/by-sa/3.0/), consistent with the CSL styles repository.
