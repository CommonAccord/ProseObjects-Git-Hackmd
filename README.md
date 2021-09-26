# Integration of Git <-> ProseObjects

* Goal

Ideally, all transactions, all events in transactions and all relationships would be expressed in a graph of semantically linked and labeled nodes, expressing or linking to all information relevant to the transaction, events and relationship.  The events, transactions and relationships would be "computable" in at least the sense that a human or a machine could easily parse them, could see history, compare to prior similars, immediately present the transaction particulars and action points, make recommendations.  Algorithmic elements could be included in the graph, fully automating those parts of the events and transactions. 

Ideally, the graph would be sliced into small pieces, each under control of the persons to whom that part of the graph is relevant, interoperable.  Each piece could be invented or adapted by participants as they see fit, but also would tend towards interoperability, consistency, folksonomies and standards.

Ideally, the same mechanisms for performing transactions would be used for framing the transactions, and also for governing relationships, and the entire system.  The system would be fractal in this dimension, too. 

This seems within reach. 

* Basic - Current
* 
Currently, ProseObjects are files, maintained in repos.

Collections of ProseObjects can be maintained in different repos, and all work together as long as naming conventions are consistent.

A person can maintain a private repo for a transaction and can clone whatever public repos they may wish to use.

The deal negotiation confidentiality structure can be implemented in git using branches - or using nested git repos.  Confidentiality trees include such things as Party1's side versus shared versus Party2's side, and recursively P1.Legal internal conversation versus the part P1.Legal delivers to P1, same for discussion within P1.Sales and P1.Engineering).

Nested git repos currently are how I manage the relationship between Cmacc-Org and repos such as crypto-governance, Open-Source-Law, etc. (I am not sure that nested repos are a documented or approved method, and they require a one-time momentary work-around of briefly removing the .git folder from the file system (and putting it back!) so the parent repo starts tracking.)

The nested repos is probably a poor-man's (lawyer's) version of git-include.  With a system of git-include, it is possible to have a transaction solution (repo) specify _all_ of its dependencies - both prose and code - in an efficient, hard-to-fudge way. (One general principle of ProseObjects should be that all dependencies are explicit - can be found by following links.)

Git commit hashes also provide a way to deal with issues of versioning and proof.  If my deal spec (term sheet) lists deal terms and references some nodes (files), and it also references the commit hash of the repo that provides the referenced files, then it is impossible (as a practical matter), to falsify what the whole deal was.  Note that this may not guarantee access to the repo, so one should assure that it will continue to be available.  For instance by cloning a copy, or by using repos that have been widely cloned or are held by a trusted organization.  Note also that the potential problem of lack of access to the repo can be a feature.  If there is a right to be forgotten, then an organization can keep only the hash, and still protect itself against a later fraudulent claim of what the terms were.

All of this is pretty much what I've been doing at github.com/CommonAccord.

* Further integration

It would be possible to express ProseObjects as git objects.  I.e., for a ProseObject node to be a git tree object, and the keys be git file objects where the filename was the keyname and the file content was the text value. I do not have any real view on what advantages this might have but note that two people whose opinions I respect seemed to think that this would have computational advantages.  It might also fit into a kind of CONS notion of each bit of incremental work being a commit that points at the prior state. 

A disadvantage of this approach is that the current file notion is very familiar and adaptable. As a non-coder (IAMAL) I can understand what a file is. I can paste the contents into an email or paste it from an email.  It behaves in familiar ways.  

But under the hood of any production system - for instance a graph database for an enterprise - it is likely that the file-oriented nodes and key=values are re-expressed as something more granular, where each key is a separate data item.

It is also likely that ProseObjects get expressed in JSON for transaction flows, in smart contracts, etc.  This is a matter of what @xmlgrrl called "punctuation".  The BrownU team did a JSON-base version, linked to here.  http://www.commonaccord.org/index.php?action=json&file=G/MarathonVC/Demo/Acme_Barbara-OReilly_Employment.md

* Related

I mentioned to Chris and Megan that I recollected that there were some who thought that the canonical format for "files" was git.  I.e., FAT, OSX, all that was foolishness, idiosyncratic implementations of a data structure.  I remembered seeing this in the https://ocaml.org/ ecosystem. https://github.com/mirage/irmin

* ProseObjects

** What we know:

The current version, done by Primavera De Filippi in a few days at a McDonald's in Buenos Aires in 2014, is perfect in concept. https://github.com/CommonAccord/cmacc-app/blob/master/parser.pl.  It takes a file, parses it for KeyName=Value  and for =[NodeName] or for "prefixed" links in the form of KeyName=[NodeName].  This creates two data structures a hash of key=values for strings and a list of key=NodeNames for links.  (See the BrownU formalism). It renders by searching for a specified KeyName (by default this is "Model.Root"), returns the value as a string and recursively expands any {KeyNames} in the returned string.  If it fails to find a KeyName it needs in the home Node, then it will consider whether any linked Node might have such a Key in it.  A Node might contain a matching KeyName if the link is not "prefixed" or if the characters of the prefix exactly match the first characters of the Key being sought. (If the prefix exactly matches the full Key name then it (logically if uselessly) returns the [NodeName] as a string.)  Keys can contain almost any character, including internal spaces, are case-sensitive, etc.  A pure string match.  The "=" defines the end of the Key. Leading spaces don't work, and trailing ones will be ignored.  The value can also contain IFAIK any character, including "=, [, ]".  NodeNames, i.e. the names of files, are fussier.  They have to be compatible with the file system (case insensitive on OSX, lots of reserved characters, etc). The only oddity of the implementation is that NodeNames cannot contain any spaces - rendering will misread the filename as ending at the space. This bug can be a feature if we think it is a bad idea to put spaces in filenames (I do so think).

When attempting to match a {KeyName} in a Value that was found in a Node linked via a prefix, then the KeyName is treated as {PrefixNameKeyName}.  Prefixes conventionally end with a '.' so this ordinarily looks like {Terms.Price} as in the example below.  But the '.' is purely a convention, and is treated like any other character.

There is a subtlety of "deprefixing".  If there is no match for "{Terms.Price}" then the prefix is removed and the search recommences for "{Price}".  This happens automatically in the Perl (indeed, I did not communicate this need to Primavera).  Where there are multiple includes (e.g. a file OurDeals.md has a link Deal1.=[FirstDeal.md], which then has the "Terms.=" link, then we could access a {Deal1.Terms.Price}.  If not found, it would look for {Deal1.Price} and then for {Price}.  For reasons that I can only vaguely formalize, this works exactly as it should.  It is needed often, internally to any seriously structured document.  In the Perl version it is done kind of backwards - first the system looks for {Price}, then for {Deal1.Price}, then for {Deal1.Terms.Price}, each time overriding any prior match. This is inefficient since a match for {Deal1.Terms.Price} should end the search, but gets the right result.

Note that one can use the same format for Comments.  Or simply type text without a key name as a Comment.  We could imagine having a # as a reserved character.  When debugging, I'll sometimes comment out a Key=Value as #Key=Value or /Key=Value.

The current Perl implementation is entirely "lazy".  It looks for a Key only when it needs the Key, and looks for a Node only when it needs it. A proper system must be lazy, since it is quite possible to have many, many Nodes linked to that are irrelevant to any particular rendering.  A Node might for instance be a list of all possible document forms, or all previous transactions.  That can be useful, but not if rendering routinely opens up all the Nodes.

The current Perl implementation has some important limitations. Most importantly, it is incredibly inefficient.  I am told that it opens each Node (file), parses then closes it for each consultation.  Since Nodes can be stacked in multiple layers, there is a fantastic amount of opening, parsing, and closing going on.  I think that is why Heroku dies on bigger documents.  It would be far better to have the Nodes read into a temporary data structure that is retained throughout the render process and supplemented by reads to the file system as links are explored for missing Keys.  I believe this is how the BrownU version works.

I hacked the Perl version to allow wrapping of each Value string in a "<span id="keyname">, </span>".  This means that each span can be deep linked to - enabling lots of features, including semantic cross-references.  Note that the "keyname" in this is the prefixed keyname. If {Terms.Price} is found as "Price=$5 in Node DealSheet.md which is linked with a prefix "Terms.=[DealSheet.md]", then it is <span id="Terms.Price">.    

It would be a huge advantage to be able to replicate the interface elements of the XWiki versio from 2010.  That involves keeping track of the Node that is the source of a Value, and (I think) the depth of the expansion (how many "<span> </span>" are we between).  That allowed a bunch of presentation and interface interactions that are fabulously useful and persuasive.  See the YouTube video of work from 2012 https://www.youtube.com/watch?v=4ZfsyTPYFIA

I've created a number of "views" that help - including ones that list the missing Keys, either as a kind of list of Key= that can be pasted to create a first draft or with a few formats for defined terms, cross-references and the like.  In each of these, I made a new version of the Perl parser.pl because I didn't know how to pass a variable to Perl (IAMAL).

* Environments

The current version works as a mini website, locally or hosted.  I use VSCode to do my work, but it is also possible to edit in a text window in the browser. (This view is absent in at commonaccord.org because bots mess with it.)

It is also possible to work via a browser on Github.com/CommonAccord/Cmacc-Org/Doc/..." or your fork of it. (Now weirdly and wonderfully, just by pushing the period key.)

It would be great to make it easier to work locally and privately. Currently, you need to have perl and php running, and clone Cmacc-Org into a local site.  I use MAMP because IAMAL.

That might be achieved by an easier install, browser plugin or whatever.

It would be very helpful to have a VSCode plug in that rendered directly from VSCode. (We had that with a variant version, and it was very convenient.)

It would be great to have a canononical implementation of the rendering engine that could be plugged into numerous environments.

I would expect that current Contract Lifecycle Management Systems would interface to, or import ProseObjects. Some of those systems are mature, well-established, but would greatly benefit from using shared (open source) document forms. 

I expect that when (if?) the system of templates is used in normal transactions, then the native interface of the transaction system will dominate and the ProseObjects will be consulted only occasionally, for reference, like ToU, Privacy Policies or other legalish documents.   


