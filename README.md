# amalgame-formats-xml

A small, **namespace-aware, read-only XML reader** for Amalgame —
pure `.am`, no C, no external package dependencies.

It parses a document into a tree of `XmlNode` elements that you match
by **resolved `{namespace-URI}local-name`**, independent of whatever
prefix the sender chose. `<D:href>`, `<d:href>` and a default-namespaced
`<href>` all compare equal once the namespace is resolved — exactly what
WebDAV / CalDAV / CardDAV / SOAP-style payloads need.

```sh
amc package add formats-xml
# or: amc package add github.com/amalgame-lang/amalgame-formats-xml@v0.1.0
```

## Usage

```amalgame
import Amalgame.Formats.Xml

let body: string =
    "<C:calendar-multiget xmlns:D=\"DAV:\" xmlns:C=\"urn:ietf:params:xml:ns:caldav\">"
  + "  <D:prop><D:getetag/><C:calendar-data/></D:prop>"
  + "  <D:href>/cal/a.ics</D:href>"
  + "  <D:href>/cal/b.ics</D:href>"
  + "</C:calendar-multiget>"

let DAV:  string = "DAV:"
let CAL:  string = "urn:ietf:params:xml:ns:caldav"

let root: XmlNode = Xml.Parse(body)

// prefix-agnostic element test
let isMultiget: bool = root.Is(CAL, "calendar-multiget")

// collect every <D:href> regardless of prefix
let hrefs: List<XmlNode> = root.All(DAV, "href")
for i in 0..hrefs.Count() {
    let h:   XmlNode = hrefs.Get(i)
    let url: string  = h.Text()       // "/cal/a.ics", "/cal/b.ics"
}
```

## API

### `Xml`
- `Xml.Parse(src: string) -> XmlNode` — parse a document; returns a
  **Nil node** (`IsNull()` true) when there is no root element.
- `Xml.DecodeEntities(s: string) -> string` — decode the five
  predefined entities (`&amp; &lt; &gt; &quot; &apos;`), `&amp;` last.

### `XmlNode`
| Method | Description |
|---|---|
| `IsNull()` | true for the not-found sentinel |
| `Local()` / `Namespace()` | local name / resolved namespace URI |
| `Is(ns, local)` | `{ns}local` equality (prefix-agnostic) |
| `Text()` / `TextTrimmed()` | concatenated text content |
| `Attr(local)` / `HasAttr(local)` / `AttrCount()` | attributes by local name |
| `Children()` / `ChildCount()` / `ChildAt(i)` | child elements |
| `First(ns, local)` | first **direct** child matching `{ns}local` (Nil if none) |
| `All(ns, local)` | all direct children matching `{ns}local` |
| `Find(ns, local)` | first **descendant** (depth-first, includes self) |

## Scope

**Handles:** elements (open/close/self-closing), arbitrary nesting,
attributes, `xmlns=` / `xmlns:prefix=` declarations scoped to their
subtree, text + `<![CDATA[ … ]]>`, the five predefined entities, and
skips the XML declaration / comments / processing instructions / DOCTYPE.

**Out of scope (v0.1.0):** numeric character references (`&#65;` /
`&#x41;` are left verbatim — protocol control XML uses the named
entities), DTD-defined entities, validation, and serialization (output
is the caller's job — building XML by string concatenation is cheap and
explicit). Mixed-content text/child *ordering* is not preserved: text is
concatenated onto its element.

## Tests

```sh
./tests/run_tests.sh /path/to/amc
```

17 cases — namespace resolution, prefix-agnostic matching, attributes,
nesting, entities, CDATA, self-closing, `Find`/`First`/`All`, prolog
skipping, and a realistic CalDAV `calendar-multiget` body.

## License

Apache-2.0.
