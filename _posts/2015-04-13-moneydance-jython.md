---
title: "Adding Moneydance Transactions with Jython"
date: "2015-04-13"
categories:
  - "software"
tags:
  - "finances"
  - "jython"
  - "moneydance"
  - "python"
  - "software"
---

A few years ago, I decided to start tracking my finances again. After slogging it through with spreadsheets and double-entry for a few months, I went looking for a software option. From my quick survey, the personal finance software market was a wasteland, especially if you're not willing to give access to your bank account to a [web-app](https://signalvnoise.com/posts/1927-the-next-generation-bends-over "I mean disruptive, next generation bendover").

In that survey, I ran across [Moneydance](http://infinitekind.com/moneydance "Also check out Syncspace, which is crazy useful"), a Java-based, double-entry, personal finance tool. It caught my eye because it claimed to be double-entry (behind the scenes), and because it sported a developer API (and with python scripting).

Sure, Java-based was a strike (especially on a Mac, which goes out of its way to be Java-hostile), but given that the scripting access probably meant that I could get my data **out** of the tool, should I ever need to do so, I went for it. I've been very happy with it in the interim.

My bank, however, not so happy. It has this tendency to stop giving me access to my transactions after 90 days (meaning I have to remember to download things). Well, guess what happened. Three times.

So I had 3 gaping holes in my transaction data. The good news was, given the PDF statements and a moderate amount of elbow grease (mainly emacs macros), I could eventually wrangle the entries into a CSV format. The bad news was, I had let my copy of Moneydance get old, and now most of the plugins (notably, the CSV plugins) didn't seem to work anymore.

So I installed the python extension, and got to work.

The python interface, as it turns out, is a jython console. The downside if this is twofold. One, the console is extremely primitive (especially if you're used to working in, say, ipython notebook). Two, jython is based on a hideously old python (2.1? Srsly?). This makes writing code a bit of a slog. (edit in emacs. Save. Switch to console. Press "load script". Read error. Repeat).

Because it's jython, however, it means the entire Java object framework of moneydance is exposed. Infinite Kind makes the javadoc available (both for the [2012-era app](http://infinitekind.com/dev/apidoc-old/), and the [current](http://infinitekind.com/dev/apidoc/index.html) version), which, along with a [single blog post by Ric Werme](http://infinitekind.com/dev/RM-NetWorth/wiki_jython.html), forms the entire set of developer documentation for python scripting in Moneydance.

Needless to say, it's a bit minimal. But, with a genuine need at hand (several hundred missing transactions in CSV format), I decided to have a go. After some [trolling of the forums](//help.infinitekind.com/discussions/moneydance-development/252-a-proper-way-to-add-an-uncomfirmed-transaction%20 "Hinting at the solution, and the possible fragility therein"), it seemed that the way to create a transaction was as follows:

- Get a handle to the root account (RA)
- Get the transaction set from RA
- Get a handle to the source (SA) and target (category) accounts (CA)
- Create a parent transaction on SA (txn)
- Create a split transaction, directing it to CA
- Add the split to the parent
- Call txnSet.addNewTxn(txn)
- Call any cleanup necessary for the UI and/or listeners.

Three hours of paranoid experimentation followed (this **is** my finances database, after all), in which I arrived at this piece of code:

```python
import time
from com.moneydance.apps.md.model import ParentTxn, SplitTxn, AbstractTxn

ra = moneydance.getRootAccount()
visa_acct = ra.getAccountByName("Visa")
misc_acct = ra.getAccountByName("Miscellaneous")

desc="test desc"
memo="test memo"
amt = 6502 # i.e. $65.02
# Create a date in epoch format
t = (2015, 4, 10, 12, 0, 0, 0, 0, 0)
secs = time.mktime( t )
txdate = long(secs*1000)

new_txn = ParentTxn(txdate, txdate, txdate, "", visa_acct, desc, memo, -1, ParentTxn.STATUS_UNRECONCILED)
new_txn.setTransferType(AbstractTxn.TRANSFER_TYPE_BANK)
txnSplit = SplitTxn(new_txn, amt, amt, 1.0, misc_acct, desc, -1, AbstractTxn.STATUS_UNRECONCILED  )
new_txn.addSplit(txnSplit)
print new_txn
ra.getTransactionSet().addNewTxn(new_txn)

ra.refreshAccountBalances()
```

Which, to the best that I can see, worked. (here, the best that I can see = I have been running moneydance for a day since importing my transactions, and nothing has yet blown up. I set a high bar, I know...)
