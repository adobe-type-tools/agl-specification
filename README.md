# Adobe Glyph List Specification
---
© 1997–2019 Adobe.

Permission is hereby granted, free of charge, to any person obtaining a copy of this documentation file to use, copy, publish, distribute, sublicense, and/or sell copies of the documentation, and to permit others to do the same, provided that:

No modification, editing or other alteration of this document is allowed; and

The above copyright notice and this permission notice shall be included in all copies of the documentation.

Permission is hereby granted, free of charge, to any person obtaining a copy of this documentation file, to create their own derivative works from the content of this document to use, copy, publish, distribute, sublicense, and/or sell the derivative works, and to permit others to do the same, provided that the derived work is not represented as being a copy or version of this document.

Adobe shall not be liable to any party for any loss of revenue or profit or for indirect, incidental, special, consequential, or other similar damages, whether based on tort (including without limitation negligence or strict liability), contract or other legal or equitable grounds even if Adobe has been advised or had reason to know of the possibility of such damages. The Adobe materials are provided on an “AS IS” basis. Adobe specifically disclaims all express, statutory, or implied warranties relating to the Adobe materials, including but not limited to those concerning merchantability or fitness for a particular purpose or non-infringement of any third party rights regarding the Adobe materials.

Adobe holds no patents on the subject matter of this specification.

*Document Version 2.9. Last updated August 21, 2019*

---
## 1. Introduction
The purpose of the *Adobe Glyph List Specification* is to describe the computation of a Unicode character string from a sequence of glyph names. This is achieved by specifying a mapping from glyph names to Unicode character strings.

The mapping is meant to convert a sequence of glyph names to plain text while preserving the underlying semantics. For example, the glyph name for ‘A’, the glyph name for ‘small capital A’, and the glyph name for a swash variant of ‘A’ will all be mapped to the same UV (*Unicode Value*). This is useful in copying text in some environments, and is also useful for doing text searches that will match all glyph names in the original string that correspond to ‘A’.

It is outside the scope of this specification to determine the set of legal glyph names. Glyph names occur in many different contexts, each having its own definition of what constitutes a legal name. This specification assumes only that a glyph name corresponds to an arbitrarily finite sequence of Unicode characters.

The specification consists of [AGL (*Adobe Glyph List*)](https://github.com/adobe-type-tools/agl-aglfn/), which provides a mapping of specific glyph names to Unicode values, along with rules for decomposing and interpreting glyph names. Because it is anticipated that this specification will be implemented in many pieces of software, and that revising all of the implementations that use it is unlikely, this specification is intended to be stable, meaning that it is never to be revised in a substantive way. In particular, it is intended that no mappings will ever be added to AGL. Also, AGL is not meant to serve as a guide to choosing glyph names for new fonts. For that purpose, [AGLFN (*Adobe Glyph List For New Fonts*)](https://github.com/adobe-type-tools/agl-aglfn/) exists (see Section 6).

This specification supports the full range of Unicode scalar values, U+0000 through U+10FFFF. It does not depend on the character repertoire of a specific Unicode version; thus, it is applicable to past, current, and future versions of the Unicode standard.

Font producers are strongly encouraged to respect this specification when naming the glyphs for their fonts. Font consumers are encouraged to follow this specification when trying to derive content from glyph names.

---
## 2. The mapping
To map a glyph name to a character string, follow the three steps below:

1. Drop all the characters from the glyph name starting with the first occurrence of a period (U+002E FULL STOP), if any.

2. Split the remaining string into a sequence of components, using underscore (U+005F LOW LINE) as the delimiter.

3. Map each component to a character string according to the procedure below, and concatenate those strings; the result is the character string to which the glyph name is mapped.

If the font is Zapf Dingbats (PostScript FontName: *ZapfDingbats*), and the component is in the [*ITC Zapf Dingbats Glyph List*](https://github.com/adobe-type-tools/agl-aglfn/), then map it to the corresponding character in that list.

Otherwise, if the component is in AGL, then map it to the corresponding character in that list.

Otherwise, if the component is of the form ‘uni’ (U+0075, U+006E, and U+0069) followed by a sequence of uppercase hexadecimal digits (0–9 and A–F, meaning U+0030 through U+0039 and U+0041 through U+0046), if the length of that sequence is a multiple of four, and if each group of four digits represents a value in the ranges 0000 through D7FF or E000 through FFFF, then interpret each as a Unicode scalar value and map the component to the string made of those scalar values. Note that the range and digit-length restrictions mean that the ‘uni’ glyph name prefix can be used only with UVs in the Basic Multilingual Plane (BMP).

Otherwise, if the component is of the form ‘u’ (U+0075) followed by a sequence of four to six uppercase hexadecimal digits (0–9 and A–F, meaning U+0030 through U+0039 and U+0041 through U+0046), and those digits represents a value in the ranges 0000 through D7FF or E000 through 10FFFF, then interpret it as a Unicode scalar value and map the component to the string made of this scalar value.

Otherwise, map the component to an empty string.

---
## 3. Examples
The name ‘Lcommaaccent’ has a single component, which is mapped to the string U+013B by AGL.

The name ‘uni20AC0308’ has a single component, which is mapped to the string U+20AC U+0308.

The name ‘u1040C’ has a single component, which is mapped to the string U+1040C.

The name ‘uniD801DC0C’ has a single component, which is mapped to an empty string. Neither D801 nor DC0C are in the appropriate set. This form cannot be used to map to the character which is expressed as D801 DC0C in UTF-16, specifically U+1040C. This character can be correctly mapped by using the glyph name ‘u1040C’.

The name ‘uni20ac’ has a single component, which is mapped to an empty string (note the lowercase ‘a’ and ‘c’).

The name ‘Lcommaaccent_uni20AC0308_u1040C.alternate’ has three components, which are ‘Lcommaaccent’, ‘uni20AC0308’, and ‘u1040C’. It is mapped to the string U+013B U+20AC U+0308 U+1040C.

Generally, multiple names can be mapped to the same string. For example, the components ‘Lcommaaccent’, ‘uni013B’, and ‘u013B’ all map to the string U+013B.

The name ‘foo’ maps to an empty string, because ‘foo’ is not in AGL, and because it does not start with a ‘u’.

The name ‘.notdef’ is reduced to an empty string by the first step, and is mapped to an empty string by the last clause of the third step.

---
## 4. Private Use Area (PUA) values
This specification supports the mapping of glyph names to strings that contain PUA values. For example, the names ‘Ogoneksmall’ and ‘uniF6FB’ both map to the string that corresponds to U+F6FB.

This specification does not include, imply, nor assume any particular PUA usage; it merely permits the naming of glyphs such that the restored character strings include PUA code points. It is up to the producers and consumers of glyph names to establish an agreement on PUA usage.

Font designers should note that establishing this agreement with users of general purpose fonts can be difficult. It is likely that not all tools manipulating character strings built from glyph names will correctly implement PUA usage, and this can lead to incorrect or unexpected functionality. It is therefore recommended, for general-purpose fonts, that all glyph names convert to strings that do not contain PUA code points.

---
## 5. Compatibility considerations
This specification has evolved over time. Please refer to Section 7 (*Document Changes*) for changes.

---
## 6. Assigning glyph names in new fonts
For glyphs that correspond to characters in the Unicode standard, it is recommended to specify names by using the ‘uni’ prefix for characters in the Basic Multilingual Plane (BMP), and the shorter ‘u’ prefix for characters in the 16 Supplemental Planes, according to the rules in Section 2.

This does not mean that fonts are invalid if they are made without using the ‘uni’ and ‘u’ prefixes for their glyph names. With one group of exceptions, all glyph names in AGL currently work in all known environments, as well as names with the ‘uni’ prefix. The exceptions are the AGL glyph names that are associated with PUA code points. These include all of the superiors and small cap names. Use of these names will, for the purpose of searching text, lead some current implementations to map names like ‘Asmall’ to the PUA value specified in AGL, rather than to the UV for ‘A’. We now recommend naming these glyphs according to the rules specified in this section. A subset of AGL that excludes the names associated with the PUA is provided by [AGLFN (*Adobe Glyph List For New Fonts*)](https://github.com/adobe-type-tools/agl-aglfn/).

If multiple glyphs in a font represent the same character in the Unicode standard, such as ‘A’ and ‘A.swash’, they can be differentiated by using the same base name with different suffixes. The suffix (the part of glyph name that follows the first period) does not participate in the computation of a character sequence. It can be used by font designers to indicate special characteristics of the glyph. The suffix may contain periods or any other permitted characters. For example, small cap ‘A’ could thus be named ‘uni0041.sc’ or ‘A.sc’.

If there are multiple variants of the same base glyph, then the variant suffixes should include zero-padded fixed-length digits so that if and when the glyph names are sorted, the intended order can be preserved. For example, if the ‘ampersand’ glyph has 23 alternate forms, they would be named ‘ampersand.alt01’ through ‘ampersand.alt23’, rather than ‘ampersand.alt’, along with ‘ampersand.alt1’ through ‘ampersand.alt22’. This rule provides only a minor convenience for font development and testing. As noted above, these glyph name suffixes do not participate in the computation of a character sequence.

This specification does not standardize any of the suffixes. Any arbitrary suffix will work for the purposes of text searching. For convenience during development and testing, Adobe uses the most appropriate OpenType layout feature name as a suffix. For example, a small cap ‘a’ could be named ‘a.smcp’, an initial form ‘a.init’, a final form ‘a.fina’, and a swash form ‘a.swsh’. If there are additional swash forms, they could be named ‘a.swsh1’, ‘a.swsh2’, and so on. The following are examples of suffixes used in Adobe’s fonts:

* a.sc (small capital ‘a’)
* T.swash (swash variant of ‘T’)
* T.begin (variant of ‘T’ used at the beginning of a word)
* T.end (variant of ‘T’ used at the end of a word)
* T.end1 (another variant of ‘T’ used at the end of a word)
* T.alt01 (first decorative variant of ‘T’)
* T.alt02 (another decorative variant of ‘T’)
* one.superior (variant of ‘one’ to be used in superscripts)
* one.inferior (variant of ‘one’ to be used in subscripts)
* one.numerator (variant of ‘one’ to be used in fractions)
* one.denominator (variant of ‘one’ to be used in fractions)
* one.fitted (proportional variant of ‘one’, used when default numerals are all-tabular)
* one.tab (tabular variant of ‘one’, used when when default numerals are all-proportional)
* one.oldstyle (proportional oldstyle variant of ‘one’)
* one.taboldstyle (tabular oldstyle variant of ‘one’)

For glyphs that do not correspond to any character in the Unicode standard, the name will not have any technical usefulness. Any name can be assigned, as long as the name will not be interpreted as having semantic value according to the rules in this document. The practice of the Type Development Team at Adobe is that if there is any useful descriptive tag for a glyph, name it accordingly, such as ‘mouse’, ‘signForSale’, ‘christmastreeBall12’, and so on. Otherwise, name it as variant of ‘orn’ (short for ornament), such as ‘orn001’, ‘orn123’, and so on.

For glyphs that represent ligatures of standard Unicode characters, there are two suggested formats for their glyph names, as follows:

1. **Descriptive** The decomposition is expressed by joining the glyph names of the standard Unicode characters, in order, using an underscore (U+005F LOW LINE). The glyph names of the characters should specify the ‘uni’ or ‘u’ prefixes and use uppercase hexadecimal digits, as described above, or with a name from AGL. For example, the ‘o f f i’ ligature should be named ‘o_f_f_i’.

2. **UV with ‘uni’ prefix** The glyph name is expressed as the prefix ‘uni’ followed by two or more sequences of four uppercase hexadecimal digits. Each sequence of four uppercase hexadecimal digits specifies a Unicode scalar value within the BMP, in order. For example, the character LATIN CAPITAL LETTER EZH WITH CIRCUMFLEX AND GRAVE, which is not in Unicode, should be named ‘uni01B703020300’, because LATIN CAPITAL LETTER EZH is U+01B7, COMBINING CIRCUMFLEX ACCENT is U+0302, and COMBINING GRAVE ACCENT is U+0300. A ligature of the glyphs named ‘T.swash’ and ‘h’ can be named ‘T_h.swash’. ‘T.swash_h’ would be incorrect because this would be interpreted as a glyphic variant of ‘T’. All glyph names are subject to a length-limit of 63 characters, and require that they be entirely composed of characters from the following set: A–Z, a–z, 0–9, ‘.’ (period; U+002E FULL STOP), and ‘_’ (underscore; U+005F LOW LINE). Some older implementations may impose a length-limit of 31 characters.

A brief review of some past implementation issues and the consequential limits on glyph names is provided in the document *Glyph Names and Current Implementations* (based on Version 1.1, dated 2003-01-31) that is provided inline below:

**Introduction** This article is strongly time-dated, as it contains comments about current implementations. Please bear this in mind if reading it much after October of 2002.

**Where and why glyph names are used** Following the naming conventions of the (no-longer-available) article *Unicode and Glyph Names* will currently enable copying text and searching text in PDF (*Portable Document Format*) documents under a wider variety of circumstances than having no names, or names that do not follow these conventions. In the era of the Internet, where many documents must be searchable in order to be useful, this is very important.

Many PDF files are made from PostScript printer files, when the original fonts referenced by the document are not available, and the embedded font data must be used. In this case, the Unicode 'cmap' table of an OpenType font is not available, and the only clue that the PDF producer may have about the semantics of a glyph is its name.

Even when the original font file is available, a glyph that does not directly correspond to a character in Unicode may still be usefully connected to a Unicode character via its name. For example, naming a decorative variant of ‘t’ as ‘t.alt’ allows a PDF producer to note that ‘t.alt’ carries the same semantics as ‘t’ for searching and other purposes.

In the future, it is expected that more products than Adobe’s own ones will support Unicode-based text string searching of a glyph, meaning that the scope of usefulness of these rules will become much broader.

For TrueType fonts, which may be missing glyph names altogether, the presence of a Unicode value for a glyph in the 'cmap' table will still cause glyphs to be correctly associated with a Unicode character in most cases. However, for glyphs that do not map from a Unicode code point, previous comments apply.

**Why is the prefix ‘u’ not yet recommended for glyphs that are encoded in Unicode’s BMP?** The prefix ‘u’ is not supported by Acrobat Versions 4 and 5. It became supported by Acrobat Version 6 and later, which is also when support for Unicode characters outside the BMP (*Basic Multilingual Plane*) was introduced. AGL names and glyph names that use the prefix ‘uni’, along with the ‘.’ and ‘_’ parsing rules, are already supported by Acrobat Versions 4 and 5.

**Length and character set limitations on names** Glyphs from Western OpenType/CFF and TrueType fonts must still be referenced in many workflows as name-keyed font data. As a result, glyph names are still subject to the length and character set limitations that are imposed by the Type 1 specification and PostScript interpreter implementations. Although both of these specify that a glyph name be no longer than 31 characters in length, in practice, particularly in modern environments, glyph names can be as long as 63 characters, but must not start with a digit or period, and must be entirely comprised of characters from the following limited set:

* A through Z
* a through z
* 0 through 9
* . (*period*)
* _ (*underscore*)

The only exception to these requirements is the special ‘.notdef’ glyph.

For example, ‘twocents’, ‘a1’, and ‘_’ are valid glyph names, but ‘2cents’ and ‘.twocents’ are not.

**ATM and kerning pair filtering** For many applications, support for kerning in OpenType fonts was provided to a limited extent by the Windows and Macintosh versions of ATM (*Adobe Type Manager*). This limitation arises because most applications that were not OpenType-aware assumed that all of the kerning pairs in a font can reasonably fit is a single table, and that there will be no more than a few thousand kerning pairs. Providing more kerning pairs than this caused such applications to crash. With class-based kerning supported by OpenType layout, even a font with only 220 glyphs would usually exceed this limit, if well-kerned. In order to allow use of such OpenTyoe fonts without crashing many current applications, ATM supported kerning via legacy OS-based APIs by first fully expanding the class-based kerning to a list of single glyph-name pairs, and then filtering this list through a hard-coded list of glyph names. If the glyph name on either side of the kerning pair was not present in the filter list, the entire kerning pair was omitted. *The kerning pair filtering list for Windows 95 and Mac OS 9, along with that for Windows NT and Windows 2000, is no longer available.*

---
## 7. Document Changes
Version 2.9 (August 21, 2019) Editorial update.

Version 2.8 (August 9, 2018) The length-limit of glyph names was adjusted from 31 to 63 characters to reflect current practices and implementations, along with a note that some older implementations may impose a 31-character limit.

Version 2.7 (August 12, 2017) The external document entitled *Glyph Names and Current Implementations*, with editorial changes, was appended to Section 6.

Version 2.6 (March 28, 2015) Minor editorial revision related to migrating it to GitHub.

Version 2.5 (November 10, 2010) Minor editorial revision related to making it an open specification.

Version 2.4 (September 24, 2003) Minor revision. Pointed URL for Adobe Glyph Names for New Fonts to a new revision.

Version 2.3 (April 17, 2003) Minor revision. Added a short sentence to make it clear that the ‘uni’ prefix can be used only with BMP Unicode values.

Version 2.2 (January 31, 2003) Minor revision. Added a link to list of glyph names to use when making new fonts, and emphasized that the AGL Version 2.0 is not meant for this purpose, nor is it about encoding glyphs in a font.

Version 2.1 (November 4, 2002) Minor revision, expanding the section on assigning glyph names in new fonts.

Version 2.0 (September 20, 2002) Major revision, which focuses the document on the conversion of glyph names to Unicode scalar values;addition of many names to the AGL; update of the ZapfDingbats list to Unicode Version 3.2.

Version 1.1 (December 17, 1998) Generally revised entire document. Updated most tables and data files. Added section on selecting glyph names. Pseudo code for extracting semantics expanded to include non-Unicode ligatures and glyphic variants. Added section on providing separate designs for double-mappings. Removed section on discrepancies with WGL4 (no longer applicable; WGL4 was updated).

Version 1.0 (November 10, 1997) First version.
