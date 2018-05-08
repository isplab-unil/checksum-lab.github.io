# Explainer: Hash-based Integrity Verification for Web Downloads

## The Problems

*   Webmasters want to make sure that the content they _reference_  through links (`<a>` elements) on their webpages (typically hosted on a content delivery network / mirror or hotlinked) is not modified. More specifically, when they include download links on their webpages, they want to make sure that the file downloaded by their visitors corresponds to what they intended to share when they included the link. For instance, a program could be corrupted to include [malware][Keydnap].

	The main solution used by webmasters is to include on the webpage a checksum (generated from a cryptographic hash function and represented in its hexadecimal form, e.g., "5cd09...") of the file referenced in the download link. To check the integrity of the files they download, visitors are expected to rely on a third-party program (e.g., shasum in the terminal) to compute the checksums of the downloaded files and compare them to the checksums included on the webpage. This practice is quite popular, but rarely used by non-expert visitors. It is used for the following software programs to name a few: Android Studio, Bitcoin, Ethereum, GIMP, GoLang, GnuPG, Inkscape, IntelliJ, OpenOffice, Plex, VLC (screenshot below), etc.
	
	![Screenshot of VLC's download webpage featuring a SHA-1 checksum](vlc.png)

	A [study][CCS] on this topic was presented at the 2018 ACM Conference on Computer and Communication Security (CCS).

*   Similarly, webmasters want to make sure that the content they _embed_ on their webpages (typically hosted on a CDN or hotlinked) is not modified. More specifically, when they include images or videos on their webpages, they want to make sure that the content seen by their visitors corresponds to what they intended to show when they included the hotlinked resource. For instance, an image could be replaced with an offensive or controversial one.

[Subresource Integrity][SRI] allows webmasters / web developers to ensure that a sub-resource (e.g., a script) will execute only
    if its content matches the specified checksum. For example, the user agent ensures that script loaded via
    "`<script src='whatever.js' integrity='sha256-...'>`" will only execute when a SHA256 hash of
    the script's content matches the specified integrity attribute. Yet, to date, SRI supports only `<link>` and `<scripts>` elements. The specification mentions potential extension to other elements including `<a>` and `<img>`:
    
    > Note: A future revision of this specification is likely to include integrity
    > support for all possible subresources, i.e., a, audio, embed, iframe, img, link,
    > object, script, source, track, and video elements.


## The proposal
The proposal is quite straightforward: extend Subresource Integrity to (at least) `<a>` and `<img>` elements. These elements would include an optional "integrity" attribute containing a checksum against which the downloaded resource would be compared.

For `<a>` elements, the solution could be restricted to elements with the "download" attribute. 

This solution has been discussed (for downloads, i.e., `<a>` elements) in [Issue 68 (WebAppSec -- Subresource Integrity)][SRI68], [Link Hashes][LinkHashes], and [Link Fingerprints][LinkFingerprints]. However, it might be too restrictive, as the "download" attribute works only for [same-origin URLs][download-same-origin] which are less problematic from a security perspective.

This also raises UI/UX questions: What happens in case of mismatch? The downloaded files could be put in quarantine or a warning could be shown. `<img>` elements could be hidden or replaced with placeholders. Security issues should be taken into account here to make sure that the mechanism used in case of mismatch does not leak information about the validity of the checksum (as discussed in [SRI v1][SRIcross] and in a [thread][SRI68]). 

A [study][WWW] on SRI was presented at the 2020 Web Conference (WWW). It shows that the adoption of SRI is modest and that developers would be interested in extending it to `<a>` and `<img>` elements. 

## Additional notes

Some websites include PGP detached signatures instead of checksums so that visitors can both authenticate the source of the file (e.g., the developer for a program) and verify its integrity. The standard could be extended to signatures as well, along the lines of [signature-based SRI][signature-based-sri].

## FAQ / Remarks

* **This would not work for dynamic content such as webpages.**

	Indeed, integrity should not be specified for dynamic content (e.g., a webpage containing a dynamic part such as the date and time) as the checksum would never match. Even if it works for a static webpage, it does not certify the integrity of the sub-resources of that page. [Signature-based SRI][signature-based-sri] would be more appropriate for dynamic resources. This questions the use of SRI for `<iframe>` elements for instance.

* **This would pose problems for URLs that point to the _latest_ version of the same program (e.g., "https://www.website.com/program/latest") when a new version is released.**

	Indeed, in this case the checksum must be updated when the content of the referenced file changes (again, [signature-based SRI][signature-based-sri] could be a solution). This is also the case for the `<link>` and `<style>` elements of [SRI v1][SRI] and for package managers such as brew. Caching would be a problem as well in this case.

	In the case where webmasters rely on content management systems (CMS) such as WordPress, the generation of the checksums and their periodic verification and update can be semi-automated by the CMS. A proof-of-concept implementation for WordPress is proposed in a recent [study][CCS] on this topic (mentioned above).
	
* **This would prevent features such as progressive loading for images and streaming for videos.**

	Indeed, as the content must be completely downloaded for checking its integrity, images could not be progressively displayed (e.g., using a as-received rendering or interlaced rendering) as done in most Web browsers.

[Keydnap]: https://transmissionbt.com/keydnap_qa/
[SRI]: https://w3c.github.io/webappsec-subresource-integrity/
[download-same-origin]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a
[SRI68]: https://github.com/w3c/webappsec-subresource-integrity/issues/68
[origin-policy]: https://wicg.github.io/origin-policy/
[CCS]: https://serval.unil.ch/resource/serval:BIB_9BD511E5C0D0.P001/REF
[WWW]: https://serval.unil.ch/resource/serval:BIB_641044F40080.P001/REF
[LinkHashes]: https://wiki.whatwg.org/wiki/Link_Hashes
[LinkFingerprints]: http://www.gerv.net/security/link-fingerprints/
[signature-based-sri]: https://github.com/mikewest/signature-based-sri
[SRIcross]: https://w3c.github.io/webappsec-subresource-integrity/#cross-origin-data-leakage

## References
Alexandre Meylan, Mauro Cherubini, Bertil Chapuis, Mathias Humbert, Igor Bilogrevic, Kévin Huguenin. **A Study on the Use of Checksums for Integrity Verification of Web Downloads. In ACM Transactions on Privacy and Security**, to appear, p. 35, 2020.

Bertil Chapuis, Olamide Omolola, Mauro Cherubini, Mathias Humbert, Kévin Huguenin. **An Empirical Study of the Use of Integrity Verification Mechanisms for Web Subresources**. In Proceedings of the The Web Conference (WWW), Taipei, Taiwan, pp. 34-45, Apr 2020. [doi](http://dx.doi.org/10.1145/3366423.3380092)

Mauro Cherubini, Alexandre Meylan, Bertil Chapuis, Mathias Humbert, Igor Bilogrevic, Kévin Huguenin. **Towards Usable Checksums: Automating the Integrity Verification of Web Downloads for the Masses**. In Proceedings of the 25th ACM Conference on Computer and Communications Security (CCS), Toronto, ON, Canada, pp. 1256-1271, Oct 2018. [doi](http://dx.doi.org/10.1145/3243734.3243746)

## Acknowledgements
This work is part of a research effort, conducted by the [ISPLab](https://www.unil.ch/isplab) at the University of Lausanne, on integrity verification for web subresources. This project is partially funded with grant #19024 from the [Hasler Foundation](https://haslerstiftung.ch/en/).

