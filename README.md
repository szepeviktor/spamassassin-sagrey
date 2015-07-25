# Mail::SpamAssassin::Plugin::SAGrey

SAGrey is a SpamAssassin plugin that provides a selective greylist tagging mechanism based on data that is already available in the SpamAssassin engine.

Greylisting is a technique used by email servers to temporarily defer email messages from first-time senders. According to the Internet email standards, systems must be able to recognize deferrals and requeue the outbound message(s) for later retransmission (at which point they would no longer be a first-time sender), and most legitimate email systems implement support for this feature. On the other hand, spam is typically sent by systems that lack the queuing resources required to implement this feature for the amount of email being transmitted. As a result, temporarily deferring email messages from unknown senders often acts as a very reliable spam filter, in that legitimate systems will usually retry the message later, but the spammers never return.

Since greylisting typically works at the protocol level, support for greylisting is also typically implemented in the SMTP server directly. However, not all MTAs implement this feature, and there are also some known problem cases with some email senders that do not properly implement the necessary queuing and retransmission feature. There are also some situations where an end-user may want to implement a lightweight tagging service for local processing, possibly because the email servers are not under their control. These are some of the areas that SAGrey is intended to address.

More s pecifically, SAGrey provides a greylist tagging service, but it does not interfere with message transfer or delivery by itself. The tagging service is achieved with a simple two-stage lookup: First, the plugin checks the final message score to see if it exceeds the user-defined spam threshold value. If so, the module then looks to see if the sender's email and IP address tuple already exist in the SpamAssassin auto-whitelist (AWL) database. If the incoming message does not meet the threshold score, or if the message sender is already known to the AWL database, the module simply exits and the underlying message is left alone. However, if the score indicates that the message is spam and the sender is unknown to the AWL subsystem, then the SAGrey module assumes that the message is one-time spam from a throwaway or zombie account, triggers the SAGREY rule, adds a user-defined score to the current spam score, and optionally creates a new header in the message. The rule name and/or header field can then be used to perform additional message-handling functions, such as having a subsequent delivery or transfer agent defer processing, or the score can be used to penalize the message if so desired.

This model has several benefits over MTA-specific greylisting mechanisms: First, it only subjects probable-spam to greylisting, instead of processing all incoming messages. Second, it reuses the existing spamassassin database of known message senders, meaning no additional databases have to be maintained. Another benefit is that it can work at the MTA level (providing that your MTA is able to defer acceptance or delivery based on the presence of header-field data), but it is not bound to the MTA and can also be implemented in delivery agents if so desired.

### Installation

To use this plugin, perform the following steps:

1.  Review the SpamAssassin threshold value, as defined by the "required_score" field in one of the SpamAssassin configuration files. Messages with a spam score that is equal to or higher than this value will trigger additional processing in SAGrey, while messages with a lower score will be ignored. SpamAssassin uses a default value of "5" if this setting is not explicitly defined.
2.  Ensure that the SpamAssassin Auto-Whitelist database is enabled and operating correctly. For informaiton about the different configuration options, refer to the [module documentation](http://spamassassin.apache.org/full/3.2.x/doc/Mail_SpamAssassin_Plugin_AWL.html).
3.  Download spamassassin-sagrey.0.2.tar.gz to a temporary directory on the SpamAssassin host system.
4.  Expand the archive with the command `tar -xvzf spamassassin-sagrey.0.2.tar.gz`, and copy the files from the `spamassassin-sagrey` directory that is created to the main SpamAssassin directory.
5.  Change to the main SpamAssassin directory, and use a text editor of your choice to review the contents of the `sagrey.cf` configuration file. By default, SpamAssassin will add the "SAGREY" rule to the main header, and add 1.0 to the message score, but no additional headers will be added to the message. The `sagrey.cf` configuration file can be modified to change these settings as desired. however it should be noted that a score of zero will cause SpamAssassin to skip the test entirely.
6.  Execute the command `spamassassin --lint` to verify that SpamAssassin operates as expected.

Once the software has been installed and configured, and incoming messages are being processed through SAGrey, any additional downstream message-handling software can be configured to use the provided data.

### Debug Output

SAGrey implements support for SpamAssassin debug output, which can be activated with the `--debug` parameter to the SpamAssassin command-line. The module-specific debug messages are marked with the string "SAGrey:" as shown in the example below:

```
dbg: SAGrey: message score indicates spam ... looking for sender history
dbg: SAGrey: sender email and host addresses seen before ... ignoring
```

Copyright © 2010-2011 Eric A. Hall.
