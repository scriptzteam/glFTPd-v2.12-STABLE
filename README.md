!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!! IF YOU RUN INTO ANY CRASH IN error.log WITH THIS BUILD PLEASE LET THE DEVELOPERS KNOW !!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
glFTPd v2.12 (2021-12-25):
Change: added support for OpenSSL 3.0 and switch version to OpenSSL 3.0.1
Change: add support for KTLS and SSL_sendfile
Change: removed support for DHPARAM_FILE option
Change: cleanup glftpd.conf parsing for pasv_addr and active_addr
Fix:	Fix infinite loop sometimes happening with load balacing multiple named interfaces
	with active_addr/pasv_addr.
Change: Make "site users" alphabetically sorted.
Change: Site change now supports "xx[m/g/t]" units for max_dlspeed, max_ulspeed, wkly_allotment, max_allot_size. For example "site change username max_dlspeed 2g".
Change: Improved installgl.sh to use "/usr/local/etc" for glftpd on fbsd.
New:	Enhance some errors in error.log to provide more info.
New:	Commandline parsing errors now stop glftpd from starting so check your syslogs if gl doesnt start :)
Change:	Dont report some ssl errors so error.log isnt so spammed with common network problems.

glFTPd v2.11a (2021-01-03):
Change: ASCII uploads are now not allowed by default! If you want to allow them for some (or all) files please add "allow_ascii_uploads" setting to your config file.
	ASCII downloads are enabled by default but you can disable them using "ascii_downloads -1 *" for example.
Change:	Returned to pre-2.11 ASCII download behaviour and dont add \r before \n in files that already have it. (For example if uploaded in BINARY mode.)
Fix:	Fixed handling of ASCII uploads that contained 0xFF character.
New:	Added "show_extension_utf8 yes" option for glftpd.conf to add UTF8 as a FEAT reply. Try this if you need unicode support for filenames/directories.
	Please note that not all possible characters might be supported so please test and report any possible issues. Remember to remove the restrictions from
	path-filter etc.

glFTPd v2.11 (2020-12-24):
Fix:	Improved ABOR handling to fix some bugs introduced in BETA3.
Change:	Increased keyword size (to 30 chars) to allow longer custom commands.
Fix:	Fixed "site show" command.
Fix:	Fixed group filtering in the stats binary with -o.

glFTPd v2.11 BETA 3 (2020-07-02):
Fix:	Fix real time CRC32 calculation on the fbsd build.
Change:	The networking code for control connection was rewritten. Should result on much less flushes when TLS is used which means
	less packets sent and much lower latency.
Change:	Enable SSL read-ahead for data connection.
Fix:	Added workaround for TLS1.3 issues with upload not being complete if client never reads anything from the data connection (or doesnt use proper SSL_shutdown()).
	The workaround is to disable TLS session ticket sending for data connection that only writes to glftpd but never reads. This however means that TLS session reuse isnt possible for uploads.
	See https://github.com/openssl/openssl/issues/7948 for details. For FTP client authors. If you care about session reuse for FXP its possible to use kind of "reverse" fxp and send the CPSV
	to the server which will be sending the data so it looks like upload to the server which does the SSL_accept(). Unfortunately this is not the default behaviour for most clients. Another option
	if to use the SSCN command together with normal PASV fxp mode and make the source act like SSL server and target as SSL client. If the client doesnt receive the TLS 1.3 renegotiation ticket
	it will still connect just fine but it needs to do the whole SSL handshake and key exchange resulting in a small performance loss.
Change:	Disabled bidirectional SSL_shutdown() by default in glftpd. With the workaround mentioned above its not needed anymore (previously glftpd was waiting for confirmation from the server that
	the shutdown was successful, which for upload/download doesnt make so much sense). This will cause issues with clients or fxp sources that dont have the previous workaround (you will get
	"Connection reset by peer") so you can enable the old behaviour with "SSL_CLEAN_SHUTDOWN 1" in glftpd.conf. In case you are worried about compatibility its probably better to use it as it only
	results in small performance loss.
Fix:	the "show_totals" line in glftpd.conf was wrong for last 17+ years and noone complained. Anyway if you want show_totals to actually work, change the second parameter to max number of lines you want to show.
Change: optimize "show_totals" feature so it doesnt follow symlinks. Interestingly enough it was doing that even tho it wasnt actually working due to the glftpd.conf bug.
Change:	Disable resuming or upload/download for ASCII transfers. Also disable SIZE command for ASCII transfers.
Fix:	Fix handling of spaces while handling SITE commands ("SITE <SPACE> <SPACE> RULES").
Change: Max FQDN hostname length is now properly set to 253.
Change: Added support for DOS line endings when reading glftpd.conf

glFTPd v2.11 BETA 2 (2020-05-03):
Fix:	dirlog is again properly adding entries
Fix:	improve mmap error handling
Fix:	Round-robin with pasv_addr with nat-enabled

glFTPd v2.11 BETA 1 (2020-03-23):
Fix:	Commented out DHPARAM_FILE line from the default glftpd.conf.
Fix:	Rare crash when adding ip.
New:	Added options to configure upload/download buffer size. See
	upload_buffer and download_buffer config options.
Change:	crc32 algorithm was changed to slice-by-16 with prefetch (https://create.stephan-brumme.com/crc32/)
Fix:	dirlog asserts when dirname was longer then 255 characters
Change:	Changed the maximum for mmap_amount and ul_buffered_force to 1000
Change: Changed the default mmap_amount to 100 (this allows memory to be reused between glftpd processes
	if some fully uploaded files are read by multiple users at the same time)

glFTPd v2.10a (2019-12-30):
Fix:	cscript was not working with an assert error
Fix:	IDNT parsing from bouncer was giving assert error on long dns entries
Change:	rewrote data connection timeout handling including the TLS handshake
	(timeout is 15 seconds during connection creation)

glFTPd v2.10 (2019-12-24):
Change:	Altered the following cookies to use the "display_speed_unit" and "display_size_unit" config options:
	A X Cw Ic IX IW iz I& IG IH Ih Ib IB IJ Ik IK Ip IP ID IS IV Iv IU Is Iq IQ IR Ix gb gc gm gn gW Gb Gs Gn Wa Wx Wy.
Change:	Last Online output now autochanges to TiB when UL or DL is over 2TiB. Changed output units to include the "i" now.
Fix:	The stats-bin segfaulted when trying to view nuketops (-n) since glftpd 2.08.
New:	Added "display_speed_unit" and "display_size_unit" config options and the %V and %Y cookies so one can choose in which units most stats are displayed.
New:	Added "t"|"T" as possible suffixes for giving or taking credits.
New:	Added "-v" aka "verbose" flag for the "site update" command to explicitly output "Skipping" lines.
Fix:	Selecting an IP with EPSV and round-robin.
New:	glftpd now always uses openssl 1.1.1+ which adds suppport for TLSv1.3.
	This required some internal changes but also some new configuration options.
Change:	ALLOWED_PROTOCOLS settings was removed and we now have TLS_MIN_PROTOCOL and TLS_MAX_PROTOCOL
	settings instead. The default is to support TLSv1.2 and higher (TLS_MIN_PROTOCOL set to TLSv1.2
	and TLS_MAX_PROTOCOL not defined).
New:	Special settings for TLSv1.3 cipher suites were added. See https://www.openssl.org/docs/man1.1.1/man1/ciphers.html
	and section TLS v1.3 cipher suites section for the possible options. TLSv1.3 uses separate glftpd.conf settings from
	older TLS versions and by default we use the openssl TLSv1.3 default ciphers. To override them use #CIPHERS13_FOR_CTRL,
	CIPHERS13_FOR_DIR,CIPHERS13_FOR_DATA settings.
Remove: Support for DSA certificates was removed. Please use just ECDSA or RSA certificates.
New:	Added new ARM/MIPS builds for ppc64 and ppc64.
New:	Added new ARM/MIPS builds optimized for various PI devices.
Change:	Increased internal security against overflows. Its likely this broke some small things so let us know.
Fix:	When file is uploaded to the server and no data are coming we check for ABOR or disconnect more often.
Fix:	Display correct user IP when DNS resolving is disabled (-n0) and a BNC is used which doesn't provide a revDNS in it's IDNT-line (thanks to steakboy7000 for reporting)
Change:	The '!' char is not allowed anymore in usernames.
New:	Use the '/' char as prefix to your username to disable the "use_dir_size" option. Can be combined with the '!' function in either order.
Fix:	The "site users" UL or DL stats aren't cutoff anymore when more than 7 digits are present in the value.
Fix:	Groupstats for NoGroup were multiplied by 10 since glFTPd v2.08.

glFTPd v2.09 (2018-12-24):
Fix:	Installer script on FreeBSD >= 10.
Change:	The -I flag now takes an argument specifying the timeout in seconds.
Fix:	Segfault when missing last arg for a stat_section in combination with the %IX cookie.
Change:	Last argument for stat_section is now optional and defaults to "no".
	Other stat_section lines with missing first or second arg are now completely ignored.
Fix:	Libcopy for FreeBSD >= 10.
Fix:	PASV and other things on FreeBSD.
Fix:	Support for both IPv6 and IPv4 on MacOS via the installer script (thanks to bonified2x for reporting and testing)
Change: Changed secure_ip to disable hostnames in the default glftpd.conf template file.
Change: Uncommented the secure_ip setting in the default glftpd.conf template file.

glFTPd v2.08 (2017-12-24):
New:	Allow IPv6 masks to be added to users.
New:	Support EPRT command with protocols 1 (IPv4) and 2 (IPv6).
New:	Support EPSV command with protocols 1 (IPv4) and 2 (IPv6) and the ALL option.
New:	Add new command "CEPR" which takes ON and OFF as arguments, or none; this enables a Custom Extended Passive Reply.
	This reply is slightly different from the RFC (2428) as this also returns the net protocol and net address in the same way glftpd-ng did.
	Such a reply is needed when using pasv_addr's different from the connection IP,
	when fxping to/from servers with a different address family than the connection IP,
	or when using an entry bouncer.
Change:	All IPs for a given hostname are now used at pasv_addr and active_addr.
New:	Added new nukedir_style method 3: the same as method 2 but now "empty" dirs are alway kept.
Fix:	Nuke output of amount of nukees and total bytesize was incorrect when files did not belong to an existing glftpd-user.
Change:	When (un)nuking, files with UID's with no matching glftpd-user are now combined under the invalid username ":UNKNOWN:" instead of the previous valid username "UNKNOWN".
Fix:	Unnuking a nuked dir without it's "nukesubdir" now always uses the default 1 instead of the multiplier from the previous nuke during that session.
Fix:	Stats-bin now correctly ignores stats from excluded groups.
Change: default ecc curve for certificates was changed to secp384r1 since secp521r1 seems to be much less supported lately.
Fix:	fixed a rare crash

glFTPd v2.07 (2016-12-24):
Change:	The path-filter setting now handles Extended Regular Expressions.
Change:	A few more site commands won't execute their post cscript when failing.
Fix:	Rewrite of glftpd.docs including several typofixes, inconsistencies,
	unlisted settings, incorrect arguments etc...
Fix:	The pasv_ and active_ports now use endpoints as well.
Fix:	Slightly incorrect sizes were shown in "site new" and the %IH and %IK cookies.
Fix:	The "Max users" cookie (%M) did not take exempted users into account.
Fix:	The oneliners and requests settings were not properly disabled when setting to 0.
New:	An example script for the pre_del_check script setting.
New:	Extra stat-cookies to be used: %GMONTHUP, %GMONTHDN, %GDAYUP and %GDAYDN.
New:	Addition of an unnuke-right alike the nuke-right in glftpd.conf.
New:	The use_dir_size setting now takes 'g' for GiB as argument too.
New:	A pre_fxp_check script can control allow_fxp now. (tittof)
Change:	The stats-bin now works for all public groups, not just primary.
	Output also doesn't cut usernames and groupnames anymore.
New:	Added option -o <group> to stats-bin to only show user or groupstats of certain groups.
	The -e and -g options now work interchangeably in both user and groupstats.
Fix:	Files matching ignore_type are now properly ignored when un/nuking.
Fix:	Output of "site users keyword", with keyword any of "nuke, unnuke, undupe, kill, kick, take, give, users"
	now gives the correct users when any of those listed -keyword permissions in glftpd.conf doesn't have
	the default flags.
Fix:	When nuking or deleting files in a path matching a "creditcheck"-setting for the logged in user,
	the ratio of the nuking or deleting user were used instead of from the user owning the file(s).
Fix:	Statline didn't show correct ratio when in a path matching a "creditcheck"-setting for the logged in user,
	when that user has ratio 0 for the current stat_section.
Fix:	The empty_dir penalty is not removed anymore from the credits of users with ratio 0.
Fix:	When changing ratio as gadmin for a user in your group, the ratio of the
	default.group file now takes precedence over the one in default.user.
Fix:	A user named "help" can now have it's settings changed.
Fix:	Tighten access control on the 'site change' command.
Change:	Add timeframe to 'site user' output and change it's appearance.
Fix:	Stats were wrongly deducted in certain cases with paths matching nostats
	when deleting or nuking.
Fix:	A group named "help" can now have it's settings changed.
Fix:	Dirlog is properly updated when a dir is deleted when -l or -L isn't present on commandline.
Fix:	Don't log NUKE, UNNUKE and TAGLINE anymore when -l or -L isn't present on commandline.
Change:	Output of 'site swho' is now also within the 80 symbols boundary if new templates are used.
New:	Freespace cookie showing amount in GiB: %X.
New:	Section cookie based on requested stats: %a.
Change:	Output of 'site stats' now includes month and day stats and the section they belong to, if the new template is used.
Fix:	Bufferoverflow when file with gibberish is present in usersdir.
Fix:	Nuking with "nukedir_style %N" now keeps same dirname.
Fix:	Unnuking of nuked dirs with a number after it (indicating a dir that is nuked when the same dirname is already nuked)
	now works in all nukedir_style cases without the need to give the number.
	Unnuking starts with the lowest numbered nuked dir.
Change:	Unnuking of a nuked dir when the same dir (not nuked) is already present now unnukes and puts a number after the unnuked dir.
Fix:	Unnuking with a "nukesubdir_style" setting where %D is missing or not the first var had incorrectly returned credits.
Change:	The "nukesubdir_style" setting does not use the %D variable anymore.
Fix:	Output of the nuker-bin now shows correct user and filecounts.
Fix:	Nuker-bin incorporates all nuke and unnuke fixes listed above, i.e. ignore_type-fix, creditcheck-fix, etc.
Fix:	Stats-bin now shows group description in grouptops.
Fix:	The 'site grpnfo' command now properly updates the group description.
Fix:	The 'site undupe' command now works properly for all files.
Fix:	Username was valid for example in renuser if the corresponding userfile was "."
New:	The privpath setting now takes '[:username:]' and '[:groupname:]'.
Fix:	The installer script handles multiple installs run via systemd.
New:	Users can have IPmask's added in the CIDR format: X.X.X.X/Y. All fields must be fully numeric!
New:	Users with ratio 0 in a stat_section and with a creditcheck-path applying on them whilst being in that stat_section will gain credits when uploading a
	file and lose when having their file deleted or nuked according to the creditcheck-ratio. Downloading is not affected by this.
New:	The statline (and 'site user' output) now show both UL and DL ratio taking into account the credticheck and creditloss config options.
	They can differ depending on which path you're in.
New:	Add cookies to show list of credits and ratios ordered by stat_section for a user, %IX and %I=.
New:	Add new config setting "nosymderef" which disables symlink dereference within given paths.
Change:	Glftpd is now built with latest openssl on all platforms. Openssl is built directly without any patches. You will find openssl binary included
	in the bin directory so you can use the features of the latest openssl.
Change: changed the default cipher to disable DSS ciphers. If you still use old DSA cert please move to RSA (if your cpu is good enough) or ECDSA.

glFTPd v2.06.1 (2015-12-25):
Fix:	Credits were not removed after deleting a file.

glFTPd v2.06 (2015-12-24):
Change: removed support for smaller built-in dhparam sizes and only support
	2048bit now (just in case someone used non default cipher and
	allowed export ciphers)
New:	it is now possible to override dhparam with a custom DHPARAM file
	using DHPARAM_FILE config setting (its not really necessary but at least you now have
	full control), if you really want one you can make one with 
	"openssl dhparam -out /glftpd/ftpd-dhparam.pem 2048"
Change: changed default cipher (see default glftpd.conf, its pretty much the
	same as before just written differently)
Change: allow_fxp logging messages now go to transfer.log instead of
	sysop.log
Fix:	fixed some buffer overflows
Change: libcopy.sh was updated to support ldconfig on newer debian (and
	clones) dists
Change: added support for rlinetd in installer
Change: fixed some strings in glstrings
Change: default permissions for some commands changed in the default
	glftpd.conf
Fix:	bigger uids/gids no longer crash glftpd
Change: added support for systemd in installer and a small readme
Fix:    http://bugs.glftpd.eu/view.php?id=26 (gl_spy cutting dir names)
Fix:	http://bugs.glftpd.eu/view.php?id=30 (ABOR returning too many
	replies)
Change: dont return error on USER command when the user doesnt exist (to avoid leaking info about
	user existance)
Fix:	XCUP instead of XCDUP according to rfc 775
New:	successful chpgrp sends an entry to sysop.log now
Fix:	several cases of userfile lockups and unreadable userfiles
Change:	new users are created with 0644 instead of the less secure 0666
	external scripts needing write access and running as non-root should be re-evaluated
Change:	listing in 'ps' on Linux now uses the name of the executed binary, much alike on BSD
Change: more strings in glstrings.bin for your pleasure to edit
Fix:	Use default rootpath for loading glstrings.bin when none present in conf
Fix:	Remove unnecessary logging of failures to open temp lockfiles
Fix:	Do not include default.* userfiles in certain stats anymore (no in no stats anymore at all)
Fix:	Disallow renaming a user to default.* or *.lock
Fix:	Remove errors and typos from the help files at ftp-data/help/ and use consistent format and style.
Fix:	Correct site fdupe help in glftpd.docs.
New:	Added gpdayup and gpdaydn site commands.
Fix:	Fixed headers of all gp* site commands and centered footers.
Fix:	site grpren now also renames default.oldgroupname
Fix:	Correct warning message for too long oneliner added.
Fix:	Credits were not removed after deleting a file.

glFTPd v2.05 (2014-12-24):
Change:	try to print details about ssl handshake errors to control channel
Change: fix output of "STAT" command
New:	Added loadbalancing for active addresses.
	It is now possible to give a max download and upload capacity to each active address.
	See glftpd.docs for more info.
Change:	Added a new extended SHM to store more info on transfers.
Change: Generate openssl version in glftpd version dynamically using openssl api.
New:	OSX instalation support for installgl.sh (tittof)
Fix:	fix detection of hostname/ip adress in pasv_addr 
Fix:	fbsd10 support in installgl.sh (tittof)
Fix:	fixed site gpad text in the header (ftp-data/text/gpad.head)
Change: disabled SSLv3 support (SSLv2 was already disabled)
New:	added new config options ALLOWED_PROTOCOLS that allow you to decide which TLS
	protocols are allowed by the server. You can specify one or more out of :
	TLSv1, TLSv1.1, TLSv1.2 (Default is to allow only TLSv1.2)
New:	added new config option nopathmatch to disable glftpd trying to match a dir
	when trying to CWD to a nonexistent dir; this made some clients malfunction
Fix:	fixed site ginfo text in the footer to be less confusing (ftp-data/text/ginfo.foot)
Change:	Updated the CRC32 algo used when calculating realtime crc to use intel optimized SliceBy8
	algo. (used when calc_crc option is enabled)
Fix:	fixed crash on OSX 10.10

glFTPd v2.04 (2014-01-26):
Fix:	Fixed crash that might sometimes happen with "site change"
Change: full-dynamic binary on linux has now ASLR enabled (-pie)
Change: removed exact glftpd version from the glinstall.sh

glFTPd v2.03 (2014-01-14):
Change:	Added large file system support compilation flags to
	glftpd/bin/sources/compile.sh
Change: Fix number formatting in "site users"

glFTPd v2.03 RC1 (2013-12-30):
Change:	Log into syslog until glftpd.conf is properly read.
Fix:	Check for too long keywords in glftpd.conf.
Fix:	Fix "site change [user] flags +ABC" to add all flags again.

glFTPd v2.02 FINAL (2013-12-24):
Fix:	Now allow gadmin to set a ratio of 0 or the default value, whatever
	the previous value was

glFTPd v2.02 RC5 (2013-10-13):
Fix:	nuker crash that showed up on fbsd
Fix:	64bit version of glftpd required 32bit ld-linux.so.2 when using
	libcopy.sh or installgl.sh

glFTPd v2.02 RC4 (2013-10-09):
Fix:	Fixed FreeBSD build to use openssl 1.0.1 from ports
Change: removed limits for mmap_amount

glFTPd v2.02 RC3 (2013-09-23):
Change: Disable compression for ssl data channel unless its a dirlist.
New:	Added support for ECDHE key exchange to make PFS work for ECC certs.
	(to make sure noone can descrypt recorded sessions if they get hold
	of servers pem file)
Change: Changed the key cert generation to generate ECDSA cert by default.
	This means the install script and create_server_key.sh.
	These are smaller, faster, cooler. However they require that the clients
	openssl is at least v1.0.1 and allow TLS v1.2 standard.
Info:	For anyone using pftp please change your sources to use
	SSLv23_client_method in tlsutil.cc. For some stupid reason i left it with
	SSLv3_client_method which is actually worse :( This will make your
	connections more secure and actually allow the use of ECDSA ciphers.
Change:	The config option for ssl certificate is now called CERT_FILE for
	all cert types. The old names are still supported tho.
Fix:	Fixed the nuker binary to stop damaging the nukelog file if built
	64bit.
New:	New glftpd.conf option nukesubdir_style to specify the style of the
	directory created within the nuked directory (for example to remove the user
	name). Check the default glftpd.conf for an example.
New:	New glftpd.conf setting called ECDHE_CURVE. Can be used to override
	the default curve (or the automatically picked curve if linked with
	openssl 1.0.2). By default glftpd uses secp521r1.

glFTPd v2.02 RC2 (2013-07-16):
Fix:    Several bugs in the installgl.sh script.
Change: TLS ciphers changed to always default to HIGH instead of MEDIUM.
Change: SSLv2 is now disabled by default.
Change: site chpgrp command has new help files (since RC1) that should also be updated if you did
        not do so already, their names are: site.help.siteop site.help.all site.chpgrp
Fix:	fixed issues with passive transfers and pasv_addr option

glFTPd v2.02 RC1 (2012-11-28):
Fix:    overlapping buffers with paths having duplicate slashes (rename bug).
New:    You can change a user's primary group with site chpgrp command.

glFTPd v2.02 BETA2 (2012-11-01):
New:    You can hide real username and groupname at file listings (hide_user_or_group).
Fix:    env variable SPEED is set properly again
Fix:    couple of random overflows here and there
Change: site VERS shows more info
Fix:    overlapping buffers in symlink handling
Fix:    leak in nuke (nicon)
Change: init ssl engine to support ssl hardware (pcsx?)
Fix:    small installer update
Change: improved file check on upload for some special cases on CIFS/NFS
        mounts
Fix:    plenty of 64bit fixes
Change: 64bit binary of glftpd is now fully binary compatible with the 32bit
        binary. Both output the same file format of dirlog and other log files.
        (other 64bit version of glftpd that were released might have had
        different output then 32bit glftpd)

glFTPd v2.01 FINAL (2005-12-25):
Change: updated script for keys generation and changed expire date to 900 days
Fix:    fixed documentation about site wipe, wipe doesnt remove files from dupefile db (hoe)
Change: when uploading to nodupechk directory pre_check (dupescript) will still be executed (hoe)
Fix:    site new . inside /site/test will not show dirlog of /site/test* anymore (hoe)
Fix:    its not possible anymore to give someone admin/gadmin and anonymous flag at once (hoe)
Fix:    "site wipe test" will now work is "test" is symlink (wipe doesnt recurse into links) (hoe)
Fix:    fixed crash with STOR/RNTO //// (hoe)
Change: installer now supports fbsd6 and is able to fix /glftpd/dev entries and added note
	to glftpd.docs for manual installing on fbsd6 (psxc)
Fix:    fixed parser errors in some special cases ("site  who") (hoe)
Fix:    fixed parsing of special numbers (negative or too big) in site change (hoe)
Change: ssl certificate generated by installer will now expire after 900 days by default
        (old value was 365) (hoe)

glFTPd v2.01 RC5 (2005-11-6):
Fix:    SECURITY ISSUE
	a user could cheat ip check with specially crafted DNS hostname, now dns hosts
	have DNS prefix in userfile instead of IP so glftpd knows which to use to do ip check,
	please edit your userfiles if you have DNS hostnames (hoe)
Change: when a script is executed from glftpd the arg0 will now specify full path the script file (hoe)
Change: moved all tls settings glftpd inetd commandline to glftpd.conf, see upgrading file for info (hoe)
Change: ssl ciphers are now more configurable, each connection type has its own ssl context (if you are
	a ftp client developer you can support ssl session reuse for data connections if you copy the
	session id from old data session to new one for speed boost) (hoe)
Fix:    fixed dl_incomplete for some cases (hoe)
Fix:    keywords in glftpd.conf are now case-independant (hoe)
Change: when script is executed stderr is redirected to stdout so its easier to see errors in scripts (hoe)

glFTPd v2.01 RC4 (only internal):
Change: glftpd on linux will now be linked dynamically to libc/libdl and statically to other libs,
        (this fixes crashes with libc loading libnss dynamic libs on some systems and also allows
	users to use optimized libc libraries), also we will provide fully dynamic build which links
	all libs dynamically so you can use optimized openssl, this binary should be prefered on
	up-to-date systems (and dont forget to update your /glftpd/lib dirs after system updates)
Fix:    fixed post-cscript scripts never being executed for custom "site commands" (hoe)
Fix:    fix more ghosts problems caused by inetd setting alarm signal to block (hoe)
Fix:    fix random crashes on site wkdn/daydn and other stats commands (hoe)
Fix:    fix ghost problem after ABORting a transfer (hoe)
Fix:    fix crash after glob()ing, (NLST *.zip) (hoe)
Fix:    fix file renaming/moving when sendfile() fails (hoe)
Fix:    fix some crashes :) (hoe)
Fix:    crash after unknown command from RC3 (hoe)
Fix:    in some cases TIMEOUT was logged to glftpd.log instead of login.log (hoe)

glFTPd v2.01 RC3 (2005-08-10):
Change:	changed the default conf to allow gadmins to use gadduser since they can be admin of multiple
	groups. (bloody)
Change: remove old algos from Passchk.c (hoe)
Fix:	Fixed a number of problems with the speed limiting, it should be accurate and finaly be
	working 100%. (bloody)
Fix:    Fixed bug in reading damaged lines in passwd/group files. (hoe&bloody)
Change: If dirlog is detected as damaged glftpd will now create new empty one. (hoe)
Change: Users IP will now not be visible in process title (ps output). (hoe)
Fix:    fixed ftpwho.c and killghost.c to support big ipc keys (daxxar)
Fix     added missing R letter to "GROUPNFO" in default groupfiles
New:    support NOOP during transfer and handle uknown commands too (hoe)
Fix:    fixed semaphore locking from RC2, you shouldnt get semop() bug anymore (hoe, thx to ofCOURSE and Ceps for help)
Change: when user is not logged in unknown commands will send "please login first" error (hoe)

glFTPd v2.01 RC2 (2005-05-22):
Fix:    fixed various IDNT parsing errors (hoe)
Change: added REST STREAM into FEAT list (hoe)
Fix:    Fixed EXEC cookie parameters (hoe)
Fix:    valid_ip wasnt working properly with more parameters (hoe)
Change: added more verbose error reporting if crash is detected (hoe)
Change: all sockets will now have KEEPALIVE enabled to detect timeouts (hoe)
New:    added dirlog caching (hoe)
	-around 2mb of new shm (shared memory) is now used to store positions of directories from dirlog file
        -this makes dirlog updates much faster and should speed up massive uploads and lower machine load
	-the cache is invalidated if external change is detected
	-the shared memory is not removed when you close glftpd, you can delete it manually if needed
	 with ipc utils
	-the ipc key for shm can be changed by using "ipc_key_dirlog" in config file, by default it takes
	 "ipc_key"+1 value
Fix:    fixed crash in RC1 when xdupe wasnt defined (hoe)
Fix:    fixed upload resuming crc codes, zipscript will now get 0 as CRC if the file is resumed (hoe)
Fix:    fixed ugly problem related to max ip limit changes in RC1, site addip/site give should not
        damage peoples ips anymore (hoe,bloody_a)
Fix:    fix "stat -l" dirlists with -password loggins (hoe)
Change: optimized behaviour when download a file which is being uploaded (hoe)
Change: rename ul_buffered to ul_buffered_force, please DO NOT use this feature
        unless you really need it, see the docs for more info (hoe)
New:    added utility libcopy.sh which can be used to copy needed libraries into glftpd chroot (psxc)
  
glFTPd v2.01 RC1 (2005-04-01):
Change:	The site wipe command will now wipe symlinks (only the link, not the destination).
Fix:	The speed limiting has been fixed again, it should be working 100% now for both user
	and global limits. Please test this! Unfortunately the SHMEM format had to be changed
	to make this work.
Fix:	Make sure the simultanious group users limit is obeyed.
Fix:	Make sure the correct gadmin flags are kept when removing/adding users from/to groups.
Fix:	Fix move's across disks on *BSD.
Change:	The internal handling of the user ip's has been changed. The max limit of ip's a user
	can have has been removed.
Fix:	Sometimes glftpd could crash during a CWD command, this has been fixed.
Fix:	Fixed the site laston arguments to work as they should.
Fix:	Fixed a botscript exec problem when unnuking dirs with special chars using the external nuker.
Fix:	Fixed some small problems with site fdupe.
Fix:	The show_diz config option needed the rights instead of them being optional as it should be.
Fix:	Changed the ident timeout value back to the old (<=v2.0RC6) value.
Fix:	It was not possible to change a groups name when only the case differed.
Fix:	Fixed the external nuker to enter the nukelog entries in the right format.
Fix:	Fixed the site give and site take commands to use the bigger credit limits that are in v2
Fix:    Timeouts during file transfers are set to 120 seconds for everyone
Fix:    Fixed security issues with zip site commands like site nfo
Change: Removed support for special password hashes used in 2.0rc2-rc3
Change: Moved lots of error logging from system syslog to our error.log
Change: Added xdupe line to default config
Change: If there is no xdupe line in config glftpd will now automatically turn on xdupe for basic file types

glFTPd v2.0 (2004-12-24):
Fix:    check return value of fclose when uploading file, somehow NFS likes to fail to close a file
Fix:    when upload error occured do silent zipscript check after the error was sent to client
Fix:	There where some problems with CHOWN when moving files across disks
Fix:	Installer updates by psxc (now it should also work with fedora core 3, which has no `which` command)
Fix:	few small typo's in the docs
Fix:	the first abor response line had a multiline-indicator which was wrong
Change:	tls cleanup
Fix:    tls errors will now go to error.log instead of syslog
Fix:    when glftpd will no longer loop it if crashes during logout

glFTPd v2.0 RC7 (2004-12-15):
Fix:	it is no longer possible to modify deleted users
Fix:	when renaming a group the default.* userfiles wont be touched anymore if they contain
	the group
Fix: 	if the data connection was broken unexpectedly glftpd could logoff the user
Fix:	when an ABOR was done on a file listings glftpd would crash
Fix:    reset online[PID].bytes_xfer before every transfer (may fix the $SPEED problems)
Change: dl_sendfile now specifies block size in KBytes !!!
        recommended and default setting is 512, 0=disabled
Fix:    glftpd will try to autodetect and fix signal blocking causing ghosts problem
        when this is detected glftpd will report warning line into error.log
        with command that caused it, please report these broken commands
Fix:	When nuking an empty dir the size was shown in MB instead of KB
Change:	XDupe info will no longer be disabled when loggin in with -password
Fix:	Files transferred in ascii-mode could sometimes cut-off at random size when some characters
	where found. This problem was introduced in RC5 with the endianness fix
Change:	We now allow multiple hidden_files entries in the config
Fix:	The site syslog/login/etc commands would say the log file was empty if the file did not
	exist and also had a multi-line-indicator which would make clients hang. We now say the
	file cant be found when it doesnt exist and also notifies when its empty.
Fix:	When one changed the credits of a user using site change the log entry in the sysop.log
	would show incorrect value's.
Fix:	The binaries for which sources are in the package have been removed, please compile them using
	the compile.sh script in the bin/sources dir.
Fix:	when changing the flags of siteop the master setting only worked for the first specified master
Fix:	when deleting siteops the master setting only worked for the first specified master
Fix:	deletion and readd of users that where not in any group was impossible
Change:	the ginfo.foot cookies where wrong they have been fixed

glFTPd v2.0 RC6 (2004-11-28):
Change:	dont log wipe's that stop on an error
New:    after successful USER command, env variable USER is set, so it can be used in cscript for PASS
Fix:	if the default ratio of a group was not 3 gadmins could change it to 3 by going to 0 then to 3
Change: dl_sendfile parameter now means size of data done in one sendfile call in mb
Change: site take/give used g/m suffix to specifiy gigabytes or megabytes instead of just -m prefix for megabytes.
Fix:	fixed a number of problems with ABOR
Fix:	zero-copy (sendfile) downloads could not be disabled
Fix:    fix ghosts when user had firewalled IDENT port, and lowered timeout to 3 secs
Fix:	the external nuker now reconises the usernames properly
Fix:    you can now have 10 stat_section entires in config
Fix:    fixed oY cookie for showing year in oneliners
Fix:    fixed linefeeds in FEAT reply 
Fix:	site ginfo was case-sensitive for gadmins
Fix:	site deluser would not work when being siteop
Fix:	Removed the false warnings or "Access denied" when issuing site delip
Fix:	Removed the outdated glupdate.c and olddirclean2.c from the gcp directory
Fix:	The error messsage that notifies a user that a file is unretrievable 
	showed the username instead of the filename.

glFTPd v2.0 RC5 (2004-10-31):
Fix:    Fixed site readd and deluser to manage only users primary group for gadmin operations
        (only users primary group is used to check for slots etc)
Change:	The default behaviour is now to use zero-copy (sendfile) downloads unless specified otherwise.
Fix:	Fixed corrupted dirlog when deleting files. (thanks to chotaire for reporting) 
Change: Added leech/ratio slots to ginfo.foot and cookie system. (thanks to deam for reporting)
Change: Botscript execution uses botscript_exec() instead of system().
Fix:	Fixed a site fdupe crash.
Fix:	On Mac OSX ascii downloads would be corrupted because of an 
	endianness problem.
Fix:    site chgadmin was unable to find existing group
Fix:    Disabled site locate in default glftpd config, its insecure, allows 
	people to search
        private dirs
Fix:	When using display options on the NLST command glFTPd would not
	flush the internal buffer which caused the data not to be sent
	to the client.
Fix:	Small doc error in the site change help file.

glFTPd v2.0 RC4 (2004-10-24):
Fix:	Make site fdupe work properly and fix the crashes.
Change: all group slot related commands now check and update slots for every
        user including siteop
Fix:    site chgrp now removes +2 flag if the user doesnt need it anymore
Change: site chgrp now updates group slots
Fix:	"site fdupe  " crash fixed.
New:	Added support for the XCWD/XMKD/XRMD/XPWD/XCUP ftp commands.
Fix:	Fixed the external nuker to use multiple lines (all returned text
	was on one line).
Fix: 	Few small doc fixes.
Fix:	The SECTION env variable was not always set right, this has been
	fixed.
Fix:	Resolved the problem with the SIZE command which would only return
	a 0 value on OSX.
Fix:	When zero-copy downloading files that where still being uploaded 
	the download would not continue downloading if the end of the file
	was reached eventho it was not yet complete, this has been resolved.
Change:	All group related commands are now non-case-sensitive.
Fix:	The external utils still had some old userfile schema's which
	would make the ADDED line and the EXPIRES line go bad.
Fix:	Added/fixed account expired message.
Fix:	If a dir was unnuked the dirlog was not updated correcly. This
	problem was found in the internal nuke command and the external
	nuker tool.
Fix:	The NLST command would not write the files to the client direcly
	when using option behind it. The output was cached and kept until
	another command which does flush was called. This has been resolved.
Fix:	Fixed stats segfault on empty rootpath/datapath
Fix:	Fixed stats segfault on invalid or unreadable ftp-data/users path.
Change: Users with access to the hideuser directive will no longer be
	able to see other hideuser users.
Change:	Users with access to -showhiddenusers will now also see users
	hidden by the hideuser directive. Previously only users hidden
	by hideinwho where shown. 
Fix:	Fixed a number of issue's with the ABOR command.
Fix:	Few small doc fixes (thanks for Chotaire for the reports on 
	this and a few more issue's).
Fix:	A small mem leak that could occur when doing plain downloads
	(as in no mmap and no sendfile) and ABOR was issued has been
	found and resolved.
Change:	The ip check that is used on doing 'site addip' has been changed
	to allow octets of 255 (used to be max 254) because some 255's
	are routable these days.
Fix:	When sendfile was enabled on FreeBSD glFTPd would keep sending
	data and never stop, this has been fixed.
Fix:	A small local buffer overflow in dupescan.c has been fixed.

glFTPd v2.0RC3 (2004-09-18):
Change:	Up and downloads done within paths that are not in the dirlog
	are no longer recorded into the lastonline log.
Fix:	Fixed a small mem leak.
Fix:	Another great installer update by psxc.
Fix:	A few minor bugfixes found when cleaning the code.
Fix: 	A few problems with the pasv_ports and active_ports settings have
	been fixed. They where probebly noticeable when using a ports above
	32767.
Fix:	The behaviour of the group allotment slots was not as expected.
Fix:	The cookie files for the monthly group stats where false.
Change:	When a groupadmin adds a user the users homedir will now be set 
	according to the default.user or default.<groupname> files and no
	longer inherit the groupadmins homedir.
Fix:	Fixed various small problems with groupfile unlocking in
	the adduser procedure.
Fix:	When using the ascii_downloads setting now only files that
	match the mask will be allowed, the docs stated this already
	but any file that didnt match the mask would be allowed.
Fix:	Gadmins could loose their gadmin flag in some cases (most common
	was when doing site give).
Fix:	When using dl_sendfile 1 resumes (REST) now works properly.
Fix:	The ABOR command did not work properly in non-TLS mode.
Fix:	Zerocopy support on FBSD should now work properly.
Fix:	When a user logged off the entry added to the laston.log was bad, 
	also the size of the stats field in the laston struct has been changed.
Fix:	The password limit on "SITE PASSWD" was still set to 8 instead of 
	the new 20 char limit.
Change: Again changed password hashing system, see updated bin/sources/PassChk
Fix:    fixed SSCN ON problem with dirlists
Fix:    fixed some password size limit related errors
Fix:    fixed ginfo cookie %[%f]gW
Fix:    site users overflow/crash
Fix:    site change user expires 2004-10-10 overflow/crash
Fix:    REST command for sendfile transfers
Fix:	olddirclean2 did not handle rootpath and datapath settings in the config the 
	right way (thanks go out to binary).

glFTPd v2.0RC2 (2004-07-10):
Change: The installer has been updated again and the sources that come with
	glFTPd have had some updates too.
New:	Added support for SSCN command as alternate way of doing SSL FXL.
	See (http://www.raidenftpd.com/kb/kb000000037.htm) for more info.
Change: From now on glFTPd uses multi hashed HMAC-SHA1 encrypted passwords
	and no longer the old DES encryption. Old DES encrypted password files will
	still be useable and upon login of a user with a DES password
	glFTPd will automaticly convert it to SHA1. This also changes
	the max password size, which is now be 20 chars.
	Please note that some scripts that control user passwords has to be rewritten
	for the new scheme. See bin/sources/PassChk for example. (compiles only on linux it seems)
Change:	When a user logs-off this will now be logged into the login.log
	instead of to the glftpd.log.
Fix:	It was possible for people with negative credits to still download.
New:	glFTPd now supports buffered uploads which should decrese disk
	access and thus be faster and less cpu intensive. This option
	can be enabled using the ul_buffered directive in the config,
	please refer to the docs for more information.
New:	From now on glFTPd supports zerocopy downloading using sendfile()
	enabling this should make downloads faster and less cpu intensive
	because no data has to be copied between userspace and kernelspace.
	Do note that the speed limiting is somewhat less direct because
	of a bigger buffer. This option  can be enabled using the dl_sendfile
	directive, please refer to the docs for more information.
	NOTE: some older linux distributions will not work with this, upgrade...
Change: All ascii downloads and file listings (NLST/STAT/LIST) are much
	faster and less cpu intensive due to internal rewrites.
Change: Major speedup for all SSL traffic
Fix:	If the connection is lost during an upload and the session times out
	the CRC code passed to the zipscript will now be 000000. This to
	prevent errors occuring when people reconnect and restart the file.
	NOTE: Please check your zipscript if it supports a zero crc value
	      if you want to still get these files tested and/or not deleted.
Fix:	The site chgrp <group> comment option did not work, this has been 
	fixed. We also added the comments to the default group.txt file.

glFTPd v2.0RC1 (2004-06-13):
Fix:	The -s option of the stats bin it used the rankings from the default
	section instead of the specified one, this has been fixed.
Change: You can now change multiple groups at once using the site grpchange
	command just like this is possible with the site change command.
Change: The installer is now capable to install glFTPd on: Linux, FreeBSD,
	OpenBSD, NetBSD and Darwin (OSX). Also some enhancements and bugfixes 
	have been done.
Fix:	The stats and credits of users can now really contain bigger value's.
New:	Added a Who Cookie which shows if the user is using ssl or not.
Fix: 	The extra help tools dirlogclean.c, dirloglist.c and killghost.c have
	been updated to use the new structs.

Change: The installer has been updated with a number of bugfixes and a few
	new things (Thanks go out to psxc).
Change:	The site laston command now uses a cookie file. (If you are upgrading
	from an earlyer version of 2.0 you have to empty your laston.log and
	copy ftp-data/text/laston.* to your glftpd dir.)
New:	2 new config options (max_ustats and max_gstats) have been added which 
	can limit the amount of results shown in the user and group tops.
	Please see the glftpd.docs for more information.
Fix:	The error about the groupfile in the error.log when adding a new user
	has been removed.
Fix:	Resume will now also work with files larger than 2gb.
Fix:	When using the stats bin the -x option did not work in combination with
	the -n option, also the -T and -M options where not working correctly.
Fix:	The unnuke log entry was missing 2 quotation-marks (").

Fix:	The stats bin has a number of problems which have been resolved.
Fix:	External script execution was broken in a number of cases.
Fix:	When one nuked an emptydir the logline would be different than on other
	nukes, this has been fixed.
Fix:	killghost.c and nukelogscanner.c where fixed to compile on FreeBSD (Thanks 
	go out to psxc).
Change:	The installer has been updated, it should now work fine with FreeBSD and
	OpenBSD and some bugs are fixed (Thanks go out to psxc).

Fix:	The stats bin would crash in some cases.
Fix:	The nuker would screwup the nukee(s) stats on some cases.
Fix:	The speedlimiting code has been fixed again. Lets hope its the last time.
Change:	Changed the site grpchange command to be case insensitive about
	the specified groupname just like all other commands.
Fix:	Fixed the site chgrp (change group) command to work like it should again.
Fix:	The external utils removed the ADDED and EXPIRES lines from the userfiles.
Fix:	Added the missing lines in the default userfile for the user glftpd.

New:	The oneliners have been moved to cookie files.
Change: group leech_slots/allot_slots now has -1 for unlimited and -2 for disabled.
Change:	The complete oneliner system has been rewritten. The oneliners are now stored
	in a binary form inside the log dir and kept for ever. The syntax of the oneliners
	option in the config also changed.
New:	Site take/give accepts -m to give/take mb's instead of kb's.
Change: The hidden_files setting now takes path where it's active as first argument.
Fix:	Fixed an undisclosed buffer overflow.
Fix:	There was still a lil hickup in the site laston command, this has now
	been fixed.
Fix:	The global speed limiting code had a few bugs, if only a global rate was
	set glftpd would crash, and some calculation errors. This has been fixed.
Fix:	The external utils have been compiled staticly again. So if they did not
	work on your system, they should now.

Fix:	The site laston command was still reading the old laston file. This
	caused the command not to work at all, this has been fixed.
New:	A new cookie (%[%s]WI) has been added to show the time since the last
	successful transfer.
Change:	The online struct has changed, the time since the last successful
	transfer is now recorded.
Fix:	The speed limiting code has been fixed.
Fix:	Fixed the code of all external tools that use/update the dirlog to use
	the new dirlog format. Also the dirlog code in glftpd itself was fixed a lil.
New:	Added docs/README.multilink (something i should of done a long time ago).
Change:	The site wipe command will no longer follow symlinks but instead will
	remove any symlinks found in the wipe path.
Fix:	The adduser command sometimes showed a groupname in the adduser reply
	eventho the user wasnt supported to be or added to the group. This
	has now been fixed.
New:	A cookie has been added to display the expire date (%I!).
New:	It is now possible to set an expire date to users. Users wont be able
	to login after this date, if set to 0 the user can login till the end 
	of days.
Fix:	The user max_dlspeed and max_ulspeed ranges dont go negative anymore.

Fix:	External utils have been updated.
Change:	The install.sh has been updated by Turranius.
New:	Users can now delete their own ip's if they have acces to -delownip.
Change:	The "SITE PURGE ALL" command has been replaced with "SITE PURGE *".
Fix:	Fixed site adduser.
Change:	The userfile ADDEDBY line has been replaced by the line ADDED. This line
	will have 2 value's behind it, first the EPOCH time the user was added
	and second the username of the user who added this user.

Change:	The external utils have been updated some what more and some bugs have been
	fixed.
Fix:	Adding users as a gadmin should no longer crash glftpd.
Fix:	The speed limiting didnt work properly, it has been changed and should
	now be faster in most cases and more accurate.
New:	You can now do an online config reload in glftpd. Do do this your config
	file will need to be inside the rootpath. To make this work you need to
	add a reload_config line in the config file which specifies the config
	relative to the rootpath. Please see the docs for more information.
Change:	By default we will now log all ip's again, you can disable this with the
	-x and -X inetd options, please see the docs for more information.
Fix:	When doing SITE CHANGE luser FLAGS +/-2 the error talked about flag 4, this 
	has now been fixed.
Fix:	The file listings should not overflow at 4294967296 anymore (please test).

Fix:	A number of small other bugs.
Fix:	The denotation of gadmins in site ginfo didnt work anymore, this has been fixed.
Fix:	The multiline indicator was missing on some chgadmin commands, this has been fixed.
Fix:	Changed the ftp-data/text/group.txt file to have the right data types and right
	info in it.
Fix:	On the site grp command we will now only check if the user is a gadmin for the 
	requested group if the user is a gadmin of this group.
Fix:	Fixed some errors in different doc files, thanks to subzero, loadet and others.

New:	Technical docs for coders and scripters have been made and added.
New:	When one tries to rename (move) a file to an other dir which is actually also
	another disk glftpd will now copy the dir/file to the new location and remove
	the old one. Previously you could only move on the same disk.
Fix:	Various problems with credits/stats because of the new lengths of these options.
Fix:	The nuker now counts files as it should again.
Fix:	On the ABOR command we will now give a multi line indicator on the first line and 
	no longer on the second line. Some clients seemed to hang because of this.
Fix:	The calculation functions for the speed limitation has been changed to be more exact
	this should make limitation work better.

New:	A rewrite of the nuker is now included with this release. The new nuker
	should be way faster and have less flaws.
Fix:	Fixed an undisclosed possible security hole in site chmod.
Change: The %Iy cookie which shows the groups a user is in will now prepend each group that
	the user is a gadmin of with an *.
Change:	glFTPd should now support big files better.
New:	If the -z option is not specified in the inetd config TLS will be completely disabled.
Change:	The SHMEM now contains a new var, its called ssl_init and it shows wheather or not
	a user is using TLS or not. The possible value's are: 0 = No TLS, 1 = TLS on the control
	channel and 2 = TLS on the control and data channel.
New:	A new cookie has been added, %I- this will show the user who added the shown user.
New:	The userfiles have a new entry in them, ADDEDBY. This will contain the user who
	added this user.
Change:	If SITE PURGE is called without any extra options it will now show the help instead
	of purging all.
Change:	We can now support hostnames within the userfiles up to 95 characters.
New:	Added the SITE FDUPE command, this will do the same as SITE DUPE but then for files
	instead of directories.
Change:	The SITE LASTON command is completely rewritten. The lastonline log file has been 
	moved from the misc directory to the logs directory and is now in binary format.
	The new syntax is: SITE LASTON [num] [-user] [=group] [type]. For more info
	please refer to the docs.
Change:	If no rootpath is specified in the config we will now assume /glftpd as the rootpath.
New:	Two new stats commands have been added: GPMONTHUP and GPMONTHDN.
Fix:	Fixed a problem where SITE STATS would crash on some weird options.
Change:	It is now possible to specify more than 1 master user in the config. Please note that
	a master can do anything to any user, including other masters.
Change:	The valid_ip setting is now able to handle multiple ips.
New:	A new cookie has been added, %Gp which will show the percentage of the total bytes
	that the user has done.
New:	New requests will now be logged to the requests.log
Change:	The REQFILLED entries in the glftpd.log will now also include the user who made the request.
Change:	When "SITE DELIP someuser 9 2" was executed but someuser did not have ip 9 but did have ip 2
	glftpd would give an error and not delete any ip. From now on glftpd will give a warning
	about ip 9 but still continue to delete ip 2.
Change:	If a new group is added without an group desciption glftpd will set the group description
	to be the same as the groupname.
Change:	The SITE GRPLOG command was removed since its intergrated within the SITE GRPCHANGE command now.
Change:	The show_diz setting in the config can now have rights attached to it. There can be only
	one show_diz file per line, the new syntax is: show_diz <file> [rights].
New:	A new config option has been added, -showhiddenusers. Users with access to this setting
	will see hidden users in the SITE WHO and SITE SWHO.
Change:	The %IX cookie has been removed.
NEW:	A number of new cookies have been added with regard to the group handling change, these
	all start with %C ... please refer to the docs for a complete list.
NEW:	From now on glftpd uses groupfiles for a part of the group handling. The idea is the same as
	with the userfiles. This affects the way glftpd works with groups/groupadmins and groupslots a lot.
	The group slots will no longer be on a "per gadmin base" but global for the group. This also
	means that the userfile layout changed somewhat (the SLTOS line was removed, and the GROUP lines
	now have a second option which specifies weather or not the user is a gadmin for this group).
	One of the new features this gives users is to have one person be gadmin of more than
	one group. It is also possible to be gadmin for other groups than the users primary.
	New commands are: 
			SITE GRPCHANGE (change group settings)
			SITE GRP (show the group info)
			SITE CHGADMIN (change the gadmin(s) for a group)	


v1.32
New:    makedir now checks for free space
New:    added pre_del_script, it's executed for rmdir/file delete after internal glftpd checks
        and right before the file/dir is really deleted, this couldnt be done with cscript
Fix:    security related fix, no details
Change: compiled with newest openssl (0.9.7c) for security reasons
Change: the non-bouncer log will now appear only if -b/-B was used, and only when connection
        is denyied, if connection is accepted you can see the info in LOGIN line
        (bouncer ip set to NONE)

v1.31
Fix:    gadmin cheating bug wasnt fixed in 1.30, sry
Fix:    symlink handling of symlinks ending with "/" when changing dirs is fixed
Fix:    unnuke of BIG dirs should show proper numbers
Change: moved LOGIN and LOGOUT messages to login.log
Change: by default ips are not logged in xferlog, LOGIN and LOGOUT anymore,
        to turn on logging add -x to your inetd commandline
Add:    added -X (not -x), if -X is added to inetd commandline, ips arent
	logged even for error messages in login.log
Fix:    site who permissions with gadmin fixed
Add:    if you use bouncer, all nonbouncer connections are logged
Fix:    cannot nuke symlink anymore
Change: dupefile now uses fcntl() locks when changing it
Fix:    fixed addip of ip with .0.
Fix:    fixes in dupe handling + xdupe stuff
Add:    whole dupechecking is rewriten, should be much faster now, uses caching
        and is logs/dupefile is automatically "compressed", glftpd will now 
        randomly (1/1000) decide to remove all entries older then your days limit
        (dupe_check in glftpd.conf) (in glftpd.log there will be a note when this happens)
	(old dupecheck file will be backuped)
Add:    support for FEAT command (rfc2389)
Fix:    TOS is properly set for passive transfers
Change: Ident timeout is now 5 seconds (was 10)
New:    The "active_ports <X1[-Y1]> [<X2[-Y2]> ...]" setting, if present                                                            
        in the config file, it will cause glftpd to select a port between X and Y                                                   
        when making active connections. Without this, the next sequential port                                                      
        is used. You should have around 10 ports per user, just to be                                                               
        safe, so if your max_users is 30, make the range at least 300. (by inferno`)
Fix:    if user was in nostats, stats of nuked users were not changed, fixed (nuker binary
        is still broken (has other bugs too, dont use it)
Fix:    denydatauncrypted now shows correct error when trying to download in PROT C mode
Fix:    group stats (gpad,gpal etc) now shows correct values if they did VERY much traffic :)
Change: moved TIMEOUT to login.log and set it to hide ip unless -x is specified
Fix:    reset binary now supports userfiles with lines bigger then 255 chars, (too many sections)
Change: LOGIN log now includes used bouncer ip
Fix:    gl will not start if you have more stat_sections than 9
New:    tagline change will now be logged into glftpd.log
New:    nuke with no multiplier specified will give error message
New:    glftpd installer will now create certificate and install it (by Turranius)
Fix:    SIGCHLD rh9 error fixed
Fix:    allow_fxp now works if you use port bnc
New:    new cookie for ginfo %[%f]gW, will show wkly allotment, loot at included ginfo.body 
        (note that %IW doesnt work in ginfo)
	
v1.30   
Fix:    Fixed security bug where gadmins could overwrite/damage files in /glftpd
Fix:    Fixed bug where gadmins could cheat with leech slots
Fix:    Maybe fixed integer overflow with site nuke and big directories (4gb+) 
	(not much tested, please report if its still broken)
Change: Small SSL change to make DH key exchange more secure
Fix:    site change <user> disabled (shouldnt be there...)
Change: Added -seen setting to make it easier to disable site seen command
Change: Added -laston setting to make it easier to disable site laston command
Change: Added -traffic setting to make it easier to disable site traffic command
Change: Added -userothers settings to give ability to set site user <user> to see
        only himself or his group if user is gadmin, if you set this to "1" only
	siteop will be able to do site user <user> on everyone	
	(also used to site stats <user>)
	
v1.29.1
Fix:    SSL FXP was broken

v1.29
Fix:	Fixed security bug with site onel where glftpd didnt switch euid back from 0.
Change:	Added SSL FXP capabilities (altho no client supports this yet).
Fix: 	Fixed a problem that caused slow ascii transfers.
Fix: 	The use_dir_size was computing bad results.
Change:	The active_addr and passive_addr settings will now start at a random ip
	from the list instead of the first one.

v1.28:
Change:	New requests will now also be logged to the requests.log.
Change:	The REQFILLED logline in the glftpd.log will now also include
	the user who requested it in the first place.
Fix:	Fixed a small exploit (undisclosed for security reasons).
Change: When one does a delete on a symlink the symlink will now be removed 
	instead of its destination.
Fix:	Fixed an exploit that existed in the "site give" command.
Fix:	Glftpd will no longer crash when using "site stats 20 <non existing section>".
Fix: 	Glftpd will no longer produce strange behaviour on some commands when
	there are no lastonline line(s) in the config.
Fix:	Glftpd will no longer crash when one does "site welcome" but has no access
	to any welcome message.
Change: Added an option to the show_totals setting so you can specify the maximum
        number of lines it should show.

v1.27:
Fix:	The ascii_downloads setting could crash glftpd when downloding files
	in ascii mode in some cases (this was introduced in 1.27beta1).
Fix:	Changed the way glftpd looks for ABOR commands during transfers so
	it works in TLS mode and with clients that do not follow the RFC's.
Fix:	Some TLS updates.
Fix	Changed some variables so the group totals will be harder to overflow.
Fix:	When a user did "site user his_own_username" he would not see the 
	user.extra file (this was introduced in 1.27beta1).
Fix:	Fixed the stats commands' syntax so it works like it should again.

Fix:	If there was an I/O error reading the file to be downloaded, glftpd
	will no longer show the stat line, so clients won't get confused.
New:	There is now an active_addr option in the config. It works similarly
	to the pasv_addr option but for active connections. This is useful
        for sites with multiple Internet connections.
	Syntax: active_addr <ip/hostname>
	You can have multiple lines. Glftpd will go through them in a round-
        robin style, so that bandwidth can be distributed more or less evenly
        among all the interfaces.
Change: You will now be able to specify multiple pasv_addr lines. When you do
	this, glftpd will cycle through them on each new passive data
        connection (transfering files).
	The syntax is unchanged (pasv_addr <ip/hostname> [1]).
Change: Glftpd will now try to locate the glstrings.bin file in
	$rootpath/bin/glstrings.bin by default, instead of
	/glftpd/bin/glstrings.bin. The inetd option to specify an alternate
	path still remains.
Fix:	A small and rare memory leak in the userfile parsing code is now gone.
Fix:    The 'site wipe' command didn't interprete the 'delete' right in the
        config file the same way as the DELE and RMD commands, so under some
        circumstances, it was possible to wipe where deletion wasn't allowed.
New:    If strings "[:username:]" or "[:groupname:]" exist in the path argument
        of the rights section in glftpd.conf (upload, makedir, delete, etc),
        glftpd will replace them with username or primary group's name,
        respectively, when it's checking those rules. Exmp:
        upload /home/[:username:]/* *
        This will allow only joe to upload in /home/joe.
Change: The speed_limit setting can now take -user =group flag permissions in
        glftpd.conf, starting with 4th argument. If no 4th argument is present,
        '*' is assumed for backwards compatibility. Only rules that match
        these permissions will be considered during transfers.
Fix:    The ifip/elseip/endifip block in glftpd.conf was buggy - it would
        perform the else part of an ifip block that was nested in the else
        part of another one when it was supposed to be skipping the else part
        of the outer block.
Fix:    Renaming groups works up to full 15 characters now.
Fix:    Renaming users fully works now - you can rename to usernames > 11
        characters. Also, ':' and '/' can't be part of new username, just like
        with "site adduser".
Fix:    When a user tries to change to a directory whose parent contains
        characters that got converted by dir_names, glftpd will now recognize
        this parent as a valid directory. This should fix problems with some
        clients that use queues, but it is only one-level deep, so if the
        grandparent is like that, cd'ing will still fail.
        The same fix was applied to the MKD command (creating directories).
Fix:    It was possible to rename files/directories to names that did not
        follow the file_names and dir_names rules.
Change: Downloading of small files will be logged to xferlog just like large
        files from now on.
Change: The 'site search' command can now take more than one search string.
        Results that match ALL strings are displayed (using logical AND).
New:    The individual stats commands (site alup, site nuketop, etc) can now
        take groupname as parameter (with = in front of groupname) to count
        only people from that group. Gadmins can do this on their own group by
        default, and anyone who has access to -grpstats in glftpd.conf
        (including gadmins) can do this for any group. Exmp: "-grpstats 1" will
        cause glftpd to only display members of Friends when a siteop executes:
        "site dayup =Friends", "site monthdn 15 =Friends", etc. When people who
        do not have access do this, all users will be counted, like before.
New:    A new site command, 'predupe', will add the argument you specify to the
        dupefile database, so when someone tries to upload that file, it will
        be denied as a dupe. Exmp. "site predupe osama_naked.jpg". Users need
        access to '-predupe' in glftpd.conf to use this command.
Fix:    Some clients, like bulletproof ftp, use full paths when uploading files
        (stor /incoming/blah.zip instead of stor blah.zip). This causes
        problems when dir_names is converting directory names, because the
        client tries to upload to the old, unconverted directory, which of
        course doesn't exist. Glftpd will now see if the directory preceding
        file name exists, and if not, it will apply the dir_names rules against
        it. This should fix at least some of these problems.
Fix:    The 'site undupe' command will work better with wildcards now.
Change: Logging of fxp transfers (turned on with allow_fxp) will now go to
        sysop.log instead of login.log.
New:    Requests made with "site request blah" will now be logged to glftpd.log
        as "REQUEST" if logging (-l or -L) is enabled. Filled requests will be
        logged as "REQFILLED".
Fix:    A small bug in symlink-processing code made it impossible to make
        directories under symlinks which point to directories in certain
        configurations.
New:    Thew %NEW# cookie can now take '.' as directory name - it will show
        new uploads in the current directory.
Change: The pasv_ports setting can now take multiple ranges (or single ports).
        The new syntax is: pasv_ports #[-#] [#[-#] ...]
	...where # is any port number. If a dash and another number follow,
        glftpd will interpret it as port range. Ports will be chosen in
        sequence, starting with first argument. If an argument is a range,
        glftpd will randomly choose ports from the range. Exmp:
	pasv_ports 10000-11000 20 21 22 23 80 110 1600-1610 35000-35050
        Maximum is 50 arguments. It makes most sense to use the biggest range
        as the first argument, since it'll be used most often.
New:    A new glftpd.conf right, filemove, controls which files can be moved
        to other directories and who has access to do it. Moving is simply
        renaming to a different directory, ex. "ren bleh ../bleh" will move
        bleh to the parent directory (ren is unix ftp client's command, not
        glftpds, use rnfr and rnto directly). Access to filemove is completely
        under the rename umbrella - you can't move if you can't rename, but you
        could rename if you can't move. Exmp: "filemove /site/incoming/* 1"
        will allow siteops to move files residing in incoming or its subdirs.
New:    A new environment variable, $SECTION, will now hold the name of the
        section user currently is in. This variable changes each time the user
        changes directories. 
Change: The post cscripts for site request and site onel will not be executed
        if user has no access to these commands or if they fail.
Fix:    Turning off requests, oneliners or lastonline in glftpd.conf will no
        longer cause glftpd to crash when a user does "site request", "site
        reqfilled", "site onel" or "site laston".
Fix:    When doing 'help' or 'help site', custom commands will only be listed
        if the user has access to them. Also, access to 'help site' now obeys
        the -help permissions just like 'site help'.
Fix:    In some cases when doing site addip, the new duplicate-checking
        feature wasn't working 100%.
Fix:    When killing ghosts with ! doesn't work (there are no ghosts) but the
        site is full, glftpd will use the same reply code used to report that
        there are no ghosts, so that clients don't get confused.
Fix:    When site is full, glftpd will send the info as an error so that
        clients don't get confused and try to log in anyway.
Change: Arguments to "hidden_files" can now contain wildcards like *, ?, [].
New:    New special color cookie, "!!", will display a single !, so to display
        "hello !there" use "hello !!there".
New:    The "creditcheck" option for glftpd.conf can now take -user/=group/flag
        permission arguments. If the user doesn't match those, glftpd will
        move on to the next rule. Path is only checked if the permissions for
        this rule apply. If you don't have any permissions, * is assumed, so
        this will make it backwards-compatible.
Change: The path in 'creditcheck' will now be matched against the full path,
        including the file being uploaded (or nuked/unnuked/deleted), so it
        will work similarly to the upload/nuke/etc rights. It used to match
        against just the directory name (without the trailing slash).
Fix:    It was possible to sneak around the secure_pass setting. This hole has
        been plugged now.
Fix:    The 'site flags username' command will now correctly work for gadmins.
New:    The -usercomment setting in glftpd.conf is now gadmin-restricted,
        just like -userextra.
Fix:    Small fixes in the TLS code.
New:    Downloading files in ASCII mode can now be restricted using the
        "ascii_downloads <size> [<mask1> <mask2> ...]" setting for glftpd.conf.
        <size> is the maximum byte size that can be downloaded in ASCII mode.
        Set it to 0 or any string (like 'disabled') to disable size check.
        <mask> is file masks which are allowed for ASCII mode. Files that
        don't match them have to be downloaded in BINARY mode. Omitting this
        setting is the same as using *, meaning all files will be downloadable.
        Exmp: ascii_downloads 20000 *.[Tt][Xx][Tt] *.[Dd][Ii][Zz]
        Note: directory listings are not affected by this.
Fix:    Transfers in ASCII mode will now obey speed_limit and individual
        max_Xlspeed settings just like BINARY transfers.

v1.26:
Change: Glftpd will now skip invalid lines in the rights section, so if you
        forget to add permissions for one right, it will just be ignored
        rather than rendering all rights below it useless.
Fix:    Several individual cookies were not using user-specified formatting
        when printing "Unlimited" instead of the actual number.
New:    A new individual cookie, %[%s]I@, will display per-user idle time.
Change: Stat commands will be able to keep twice as big integers in some cases.
Fix:    Adding users with 'gadduser' and using a private group as groupname
        will not be allowed anymore.
Change: You don't need to have a '*' at the end of user IPs anymore. If the IP
        ends in a '.', glftpd will now assume '*' is behind it, so it will work
        more like /etc/hosts.allow and other IP admin software.
Change: The 'secure_pass' setting can now be specified several times for the
        same set of users. See the notes and warnings for secure_ip below.
Change: The 'secure_ip' setting can now be specified several times for the same
        set of users. In other words, fist match will not end the search,
        instead, all secure_ip lines will be examined. This allows you to have
        several alternative secure settings, like "2 numbers and an ident or
        3 numbers and no ident or hostname and ident". NOTE: this might break
        your existing rules if you have restricted rules first followed by a
        general rule for everyone. For example, if you had these two rules:
        'secure_ip 3 0 1 =lamers' and below it 'secure_ip 2 0 1 *', users in
        group 'lamers' will now match the second rule. You need to change it
        to 'secure_ip 2 0 1 !=lamers *' to make it work like before.
Change: The 'shutdown' setting will now take multiple arguments, each of which
        can be either a flag, a username with a dash in front, or a group name
        with a = in front. Users who match any of these will be allowed to log
        in, everyone else will be shown the standard shutdown message and
        turned away. "shutdown 0" has the same meaning as before, and is the
        same as "shutdown *" (no one is denied login). "shutdown 1" also has
        the same meaning as before, "allow only siteops to log in". To deny
        everyone access, use "shutdown !*". To allow only one user on, use:
        "shutdown -yourname !*".
Fix:    Glftpd will now switch to normal user before it tries to load the
        userfile of the person logging in, so that if the userfile's
        permissions are wrong, you will know right away, won't be able to log
        in, and thus won't mess up other stuff that depends on userfiles being
        readable by normal users.
Change: Nuking 0-byte files won't substract from files uploaded anymore, so
        the ratio of uploaded files to bytes will be more accurate.
Fix:    Creating directories in non-existing paths won't cause glftpd to create
        those paths instead anymore.
Change: Users who time out due to excessive idling will be logged to glftpd.log
        as 'TIMEOUT' instead of 'QUIT'.
Change: The pasv_port setting can now go beyond 32000 up to the highest
        unsigned integer value of 65536.
Change: The config settings 'alias' and 'cdpath' will no longer need '/homedir'
        in front of the directory name (as the docs suggest, but implementation
        told a different story). So if a user with homedir '/site' types 'site
        alias' and sees 'alias test /bleh/test', the actuall path that glftpd
        will use will be '/site/bleh/test' when the user types 'cd test'.
Change: Paths for stat_sections can now contain multiple components, separated
        by '[:and:]', so one section can span multiple directories. Exmp:
        stat_section LINUX /slackware[:and:]/debian[:and:]/redhat yes
Fix:    The function that validates user-specifies paths has been completely
        re-written to make it faster, more reliable, and to fix some bugs,
        such as the one that was causing deletion or renaming of symlink
        destination instead of the symlink itself.
Change: The changelog was split into changelog.old and changelog at v1.18.
Change: The no_ident glftpd.conf setting has been merged with the secure_ip
        setting. It is now a third parameter to secure_ip, and is called
        "need ident". If set to 1, the user will need a valid ident for the
        IP being added.
New:    Max length of custom site commands' output changed from 999 to 9999.
        This is also configurable now - add "max_sitecmd_lines X" to the config
        file, where X is the maximum lines you want shown.
New:    If the HOMEDIR line in default.* files contains the string "$gluser",
        it will be replaced with the new user's name, so specifying:
        HOMEDIR /home/$gluser
        will cause each new user coming from that default.X file to have a
        homedir in /home/name. So if you add user foo, it'll be /home/foo.
New:    Settings in glftpd.conf can now be conditional - depending on the IP
        mask of the person connecting. 3 new keywords, "ifip", "elseip" and
        "endifip" make this possible. The "ifip" setting takes one or more IP
        masks. If the connecting IP matches one of those masks, settings that
        follow (on lines below) will be used, until "elseip" or "endifip" is
        encountered. If the IP does not match any of the masks, settings that
        follow will be ignored. If "elseip" is encountered, settings that
        follow it will be used instead. Nesting in the elseip part is ok,
        but I wouldn't recommend nesting in the if part, it might screw up.
        Examples: this was mainly created for the pasv_addr setting, so that
        people connecting locally could have a different pasv_addr than people
        connecting from the Internet, but it can be used with any settings:
        ifip 192.168.* 10.*
          pasv_addr 10.10.10.1
          sitename_long MyLanSite
        elseip
          ifip 127.0.0.1
            include /glftpd/etc/myspeciallocalsettings.conf
          elseip
            pasv_addr 128.1.2.3 1
            sitename_long MyInternetSite
          endifip
        endifip
Fix:    A very lame bug crept into the code that handles uploading of files,
        so 0-byte files were being created when data connection couldn't be
        established.
New:    Environmental variable $HOME will now store the user's homedir.
New:    When adding user IPs (site addip), glftpd will now match the new IP
        with the existing ones, and automatically delete the ones that match
        the new one - so if you have *@1.2.3.4 added, and you add *@1.2.3.*,
        the first one isn't needed - and glftpd will auto-delete it.
Fix:    Userfile will now be reloaded before each download, so that people
        logged in more than once don't download more than their credits allow
        them. This is not serious, people could only download 1 file this way,
        and their credits would go into negative. This fix will prevent that,
        so that their credits will never go below 0.
Change: The show_diz and show_totals settings in glftpd.conf can now take
        multiple arguments, so you don't need one-per-line anymore.
Change: The pre_check script (dupescript) can now be defined more than once
        in glftpd.conf. An additional argument specifies path to which to
        apply this script - if it's missing, * is assumed. First match wins.
        Same thing for pre_dir_check.
New:    A new user option, idle_time, which specifies the default AND the
        maximum idle time the user can have. This setting overrides the
        defaults, set with -t and -T on glftpd's command line. If idle_time
        is set to -1, it is disabled, and the glftpd will work the way it used
        to. If idle_time is 0, the result is the same as having the idler flag.
Fix:    The 'mdtm' command can now work with full paths, not just files in
        current directory.
Fix:    The 'site wipe' command didn't log wiping of files, only directories.
Change: 'site wipe -r x' is now "WIPE-r" instead of "WIPE -r" in glftpd.log.
Change: 'site msg' is now completely controlled by the -msg setting in
        glftpd.conf (before, -msg only controlled sending messages, and there
        was no way to deny reading of messages). Also, 'site msg =group', 'site
        msg { user1 ... }' and 'site msg *' are now controlled by -msg too, in
        addition to being controlled by -msg*, -msg= and -msg{.
New:    The 'site stat' command can now be controlled via glftpd.conf using
        -stat -user|=group|flag. This ALSO controls displaying of the statline
        after a transfer/listing -- people who don't have access to -stat will
        get a standard "Transfer complete" response.
New:    The 'site color' command can now take an argument - 'on' will turn it
        on, 'off' will turn it off, and any other word (like show) will display
        the current setting. If no arguments are used, the setting will be
        toggled, like in older versions.
Fix:    A signaling bug could cause sessions of some anonymous users to not die
        when their connections is lost after uploading.
Fix:    Several small bugs concerning the TLS version of glftpd were fixed.
Change: The 3 zip-related site commands (nfo, ziplist, zipchk) have been
        removed. However, dn made 3 sh scripts to replace them. The scripts
        are in the glftpd/bin directory, refer to them or glftpd.docs if you
        wish to enable them. The '-ziputils' config option is also removed.
Fix:    The pwd command can no longer use color codes. This will result in
        reporting the actual directory name if there are ! characters in it,
        instead of converting !X to a color code (or just eating it if you
        have color turned off).
Fix:    The 'site user' command would cause glftpd to crash if sitename_short
        wasn't defined in glftpd.conf.
Change: Custom post scripts will no longer be executed if the command fails,
        for example if the user isn't allowed to execute it. This applies to
        the following commands: DELE, RMD, STOR, APPE, STOU, RETR, CWD, CDUP,
        RNFR, RNTO, MKD, PASV, TYPE; and the following SITE commands: CHANGE,
        ADDUSER, GADDUSER, DELUSER, NUKE, UNNUKE, WIPE, KILL, KICK.
        This list might grow in the future.
Change: If a custom script that runs before a command (pre cscript) fails, as
        where it exits with 1, post cscript for this command will no longer
        be executed.

v1.25:
Fix:    "site wipe -r " will no longer wipe current directory.
Fix:    "site grpadd name description" caused description to be corrupted when
        it was shorter than name.
Change: Gadmins will no longer be able to change ratio of siteops or user
        editors who are in their primary group.
Fix:    The "site chmod" command didn't work if you used full pathname (like
        'site chmod 777 /incoming/bleh'), it only worked with relative names,
        like 'site chmod 777 ./bleh'.
Change: If you fill your own request (site reqfilled), glftpd will no longer
        send a notification to you.
Fix:    Gadmins were able to cheat and increase their group and leech slots.
Fix:    A bug both in glftpd and in glupdate was causing some minor dirlog
        corruption - comparing directory names was causing overwriting of
        entries that did not really match.
New:    New config file option to enhance and replace "freedl". The syntax is:
        creditloss <multiplier> <leechers yes/no> <path> <permissions>
        Multiplier is multiplied by file size to get the amount of credits
        lost; second option, if set to "no", will not allow users who have
        "leech" in that directory (their ratio is 0) to download from it; path
        is the path/filename to apply this to; and permissions are the usual
        -user =GROUP flag. You can have many creditloss lines - the first path
        match stops the search. Exmp: "creditloss 0 yes /site/freestuff/* *"
        is the equivalent of "freedl /site/freestuff/* *".
Fix:    When resuming/appending a file, if the data connection can't be made,
        glftpd will no longer delete the file being resumed/appended.
Change: The free space cookie will now be initialized before welcome message
        is displayed, for those who want to use it in there.
Fix:    Viewing logs by the number of days back since today (site syslog 20
        for example) had many problems, like ignoring the year (so entries from
        sep 1 2000 would show if you did 'site syslog 5' on sep 2 2001),
        no cross-year check (so in january you couldn't see entries from
        december), etc. It has been re-written to use a much better (and
        simpler/faster) method of checking the dates, and the number of days
        can now be more than 30, so 'site syslog 366' will list all entries
        for the last 366 days, regardless of current date.
Fix:    The "time on today" variable will no longer display more than 24 hours
        if a user logged in more than 24 hours ago.
Fix:    It was possible to check if files/directories existed outside of the
        homedir using the STAT command.
New:    The killghost program was enhanced to kill more ghosts.
Fix:    A bug was introduced several versions back (not sure where) that would
        cause an error when using the NLST command with options right after a
        download. Linfxp was the only client affected, afaik.
Change: Users will no longer be able to list directories using their homedir
        as the beginning of the path. For example, if homedir is /site, a user
        could do "ls /site" and it would return the same results as "ls /",
        but it served no purpose and could be a way to guess what the homedir
        actually is - normally homedir is invisible.
Fix:    A bug was introduced in 1.25 causing users with homedir of / to not be
        able to list any directories other than /.
Change: When using "site users", the summary line at the bottom will not count
        all deleted users, just the ones that match the search, so
        "site users siteop" will only count deleted siteops.
Change: "site uchanges" is now "site syslog"; the permission setting in the
        config file is "-syslog" instead of "-uchanges".
Fix:    Some clients caused glftpd to use timeout of 60 seconds instead of
        900 or what follows -t on command line, if the user did not do anything
        after logging in.
Fix:    The reset binary could mess up userfiles greater than 4096 bytes (which
        is very unlikely to happen), and glftpd and nuker could mess them up if
        you added many stat sections at a time, like going from 1 to 10. Also,
        it's possible adding new users could be buggy if there are many stat
        sections. All should be fixed now.
Fix:    The internal nuker would sometimes log too many nukees to nukelog if
        someone nuked more than one directory in the same ftp session.
Fix:    When nuking a directory with > 2147483648 bytes of files in it (about
        2 gigs), the integer would overflow, resulting in stats and credits
        being added instead of subtracting. The limit now is 2*2147483648,
        and overflow will not add credits, it will just subtract less.
Fix:    The "free_space" option in glftpd.conf incorrectly checked free space
        for the current directory instead of the upload directory, so being in
        / and uploading /incoming/file.zip checked for free space in / instead
        of /incoming.
Fix:    In dir matching, doing "cd 3d" was the same as doing "cd 3", making
        glftpd jump to 3rd newest directory. It will now check so make sure the
        parameter is all numeric before jumping to dirs in 'site new'.
Fix:    The "site grpadd" command occasionally allowed people to create a group
        with the same name as a private group under some configurations.
Change: The "site users" command now requires a = in front of a group name to
        list users belonging to that group. Also, you can now specify a flag to
        list users that have that flag. "site users help" will now display
        file ftp-data/help/site.users.
New:    A new option lets siteops disable fxp (transferring files to/from an IP
        address that is different from the user's connection's address). The
        syntax is the following: allow_fxp yes/no yes/no yes/no permissions
        where the first setting controls downloads, the second uploads, and
        the third, if set to yes, will log all attempted transfers to foreign
        addresses to login.log. Permissions are the familiar X (flag), -user
        or =group. First permission match decides which rule is active.
        Exmp: "allow_fxp no yes no =Leech 8" will allow people in group Leech
        or anonymous users to upload from foreign addresses (like other ftp
        sites), but will not allow them to download to foreign addresses. All
        other users are allowed both, since defaults are: yes yes no *
Fix:    A small problem with shared memory was fixed, which might have been
        responsible for the once-in-a-while screwed up transfer display in
        "site who" and erroneous transfer statistics.
Fix:    The fix to 'site ginfo' in 1.24, to make it case-insensitive, did not
        work for group admins. It does now.
New:    There was delete and deleteown, rename but no renameown. There is one
        now. The new renameown right for glftpd.conf controls who can rename
        their own files/directories and where.
New:    There is now a way to hide files/directories from directory listing.
        The config variable is hidden_files, and it takes a list of
        file/directory names separated by spaces or tabs. Exmp:
        hidden_files .message private_dir
        The above will hide any files/dirs called ".message" or "private_dir".
Fix:    When resuming downloads, glftpd will now check if the user has enough
        credits to download just the amount of file he's resuming, not the
        whole file.
Change: path-filters with second field, group, being something other than '*'
        must be above the one with '*', because from now on, first match wins.
        Also, group match will happen if any of a user's groups match the
        second field, not just his primary group.
New:    "site emulate user" will load user's userfile into your process'
        memory, essentially becoming that user (although some things, like
        home directory or 'site who' display, won't change). Need -emulate
        permission in config file.
Change: The IP list that can be used as the first IP entry of any user can
        contain comments now, to clarify what the IP address is. Simply place
        the # character at the end of the IP (you can use spaces or tabs
        between the IP and the # char). Exmp: user@1.2.3.4  #user's home IP
Fix:    As I was going through most of the code to take out the hard-coded
        replies, I fixed many small bugs and optimized some code.
New:    Most glftpd responses, which used to be hard-coded in the program, are
        now kept in an external file that glftpd will load into memory when a
        user connects. There is a new directory in /glftpd/bin/sources, called
        glstrings, which contains a text file with the replies, glstrings.txt,
        along with tools to compile that into glstrings.bin (which glftpd reads
        on connect) and to de-compile an already compiled .bin file. Glftpd
        will by default try to read this file from /glftpd/bin/glstrings.bin;
        if you want to place it elsewhere, you need to use the -s option on
        glftpd command line. Exmp: glftpd -l -o -i -s/glftpd/mystringsfile.bin
        This will allow people to modify most glftpd replies, including their
        format, color, and content. It will also allow glftpd language packs.
        If you wish to mess with this, be sure to read the README file.
Fix:    A bug in site change could allow siteops to get root privileges (which
        isn't as bad as it sounds, but it might have caused some screwups
        with glftpd's permissions). This could hardly be exploited, since
        siteops already have as much privileges as it's possible to get,
        and this wouldn't let anyone get a shell or anything like that.
Fix:    A small bug with the QUIT command was confusing some clients.
Change: The dividerline config setting is now obsolete; remove it from config
        file if you have it. The line is now taken from the glstrings.bin file.
Change: The words "post" and "pre" for cscripts don't have to be lower case
        anymore.
New:    A new kind of cookie that works like an if/else statement. Let's call
        it a conditional cookie. It will check whether the user has flags or
        is in groups listed, and will print different chunks of text based on
        the result. Syntax: %?X,=group:text1:text2:? where X is a flag, text1
        is the text printed if user has access, and text2 is printed if not.
        One flag or group is the minimum condition; multiple flags or groups,
        which can be mixed, must be separated by commas. The text1 and text2
        fields can be plain text, regular cookies, color cookies, or the
        %LOGOUT cookie. The ':' character must not be used as part of text1 or
        text2, since glftpd would confuse it with the field delimiter.
        Example: %?1:You are a siteop, %[%s]Iu:You are not a siteop:?
        (this will print "You are a siteop, username" if user has flag 1, and
        "You are not a siteop" otherwise).
        Example: %?=disabled:Your account is disabled. %LOGOUT::?
        (this will log out users who are in group 'disabled', and will do
        nothing for everyone else).
        Example: Your comment is %?1,=STAFF:%[%s]I$:hidden:?.
        (this will print "Your comment is <comment>." for siteops and users in
        group STAFF, and "Your comment is hidden." for everyone else.
New:    A new glftpd.conf command permission setting, -requestadd, will control
        who can add requests. The old -request setting now only controls who
        can view them and who can use 'site reqfilled'.
New:    The 'site onel' command is now controlled by a new command setting,
        -onel, instead of -info. Adding oneliners is controlled by -oneladd.
Fix:    If PASV is listed as an idle command in the config file, glftpd will no
        longer reset the idle timer when a user issues it.
New:    The "site ginfo" command now uses cookie files ginfo.head, ginfo.body
        and ginfo.foot from the ftp-data/text dir. The new cookies are
        listed in the docs.
New:    The "site flags" command now uses a cookie file called flags.txt that
        should be in ftp-data/text. The cookies for each flag are simply
        [%s]fX where X is the flag. The cookie will print * if the user has
        the flag and a space otherwise.
Fix:    Nukees' total bytes won't overflow anymore when there are more than 2
        gigs' worth of files in a nuked dir. Same fix for show_totals.
Fix:    Downloading 0-byte files caused statline to be displayed twice, which
        screwed up some ftp clients.
Fix:    A fix to prevent zipscript from running twice when user lost connection
        didn't work completely. Another fix appears to correct that.
Fix:    Another bug in last-online code was sometimes causing an infinite loop,
        leaving ghosts that ate vast amounts of CPU.
Change: The "-noaccess" setting is no longer being used; remove it from your
        config file.
Fix:    glftpd processes on freeBSD are now named correctly:
        (glftpd: hostname: username).
New:    gl_spy now shows total upload/download speed at the bottom of the
        screen.
Fix:    Code for handling errors while uploading a file should be more
        client-friendly now.
Fix:    Adding usernames >= 24 chars was causing truncation in passwd file and
        not in userfile's name. You can't add usernames >23 chars now.
Fix:    The stats binary wasn't skipping bad userfiles, causing repeating of
        the previous user's entry in the listing.

v1.24
Fix:    Custom commands can now have custom deny-messages in ftp-data/misc/ad,
        similar to normal site commands, but the file should be called
        "custom-yourcommand", not just "yourcommand".
Change: Deleting directories will check the delete rules a little differently
        now, matching the directory to be deleted against the delete rules, not
        just the directory's parent, like before. This will allow creating more
        specific deletion rules. Exmp: "delete /site/blah*/* *" will allow
        everyone to delete files/directories in /site/blah, as well as
        /site/blah itself, or /site/blah123, etc. You might want to change
        your old rules, if you have "delete /site/????/* *" but you don't want
        people to delete /site/????, add "delete /site/???? !*" above.
        The same applies to the deleteown right.
New:    I wrote a simple C program to kill ghosts described below. Just compile
        it: cc -o /glftpd/bin/killghost /glftpd/bin/sources/killghost.c
        and run it from crontab every 5 minutes or so (as root of course):
        "0-59/5 * * * /glftpd/bin/killghost >/dev/null" should do it.
Change: glftpd will now reset the last-action time, stored in shared memory,
        even if the user issues an "idle" command, like noop. The other timer,
        which keeps track of the last meaningful non-idle command, will do the
        disconnecting. If you have scripts that check the first timer and
        disconnect people for idling, they will still be useful because on
        occasion the second timer fails to disconnect the user for an unknown
        reason. This only happens on some systems (mainly redhat with stock
        kernel) and only if the user is NOT issuing any commands.
Change: "site change user max_dlspeed" now takes an argument in kilobytes
        instead of bytes, so "site change me max_dlspeed 20" will change me's
        maximum download speed to 20K/s. Same with max_ulspeed. The number in
        the userfile is still stored in bytes, so no conversion is necessary.
Change: Non-exempt users can now set their idle time below the site "minimum"
        with "site idle". If they want to be disconnected for idling sooner,
        why stop them? The minimum idle time is now always 1 (well, it's 0 for
        exempt users). The -t setting now controls only the default idle time.
New:    New feature, xdupe, will allow clients that keep queues of files to be
        uploaded to quickly remove files that already exist on the glftpd site
        from their queue without the need to refresh. This should minimize the
        number of 'dupe' errors as clients try to upload files that have
        already been uploaded by someone else to the current directory. When
        this is turned on (through 'site xdupe'), glftpd will give a list of
        files, preceded by the string "X-DUPE:", right before the usual "this
        file is a dupe" message. The files are first matched against a list
        of file masks on the "xdupe" line in glftpd.conf. See the included
        x-dupe-info.txt for more details. Currently, pftp is the only client
        which supports this feature.
Change: Incomplete downloads, caused by user abort, lost connection, or other
        reasons, will now subtract credits from users. Resumed transfers will
        still subtract credits, but only for the amount transferred, not for
        the whole file.
New:    Upload resuming is now controlled by the "resume" right in glftpd.conf.
        The syntax is the same as overwrite. Overwrite will now only control
        overwriting files, with credits given for each file.
Fix:    When resuming uploads, stats/credits should now be handled much better:
        users will receive them for incomplete files (provided zipscript will
        accept them), and file counters will not be incremented when resuming.
Fix:    Fixed a bug with aborting uploads that was confusing ncftp and perhaps
        other clients as well.
New:    The "ignore_noop" setting is obsolete - remove it from your config
        file. New setting with similar function is "idle_commands", which takes
        a list of commands that will not be counted as being active. For
        example, "idle_commands noop pwd" will disconnect a user for idling
        even if he issues "noop" or "pwd" every few seconds. You can use
        wildcards with these commands, so cwd* will match both "cwd" and
        "cwd .". If a user issues a non-existing command, like "site woooba",
        his idle time will not reset, so it should be possible to prevent most
        clients from idling. "idle_commands" can take multiple arguments, and
        you can have as many lines of it as you want. You can use [:space:] for
        multiple-word commands like "site who" (site[:space:]who).
Fix:    "site msg *" will no longer send the message to default.user,
        default.groups, directories, and other unreal users.
Change: When nuking dirs and the same directory was already nuked (and you are
        using nuke style 1 or 2), the nuked-directory will be named
        nuked-directory.2 or .3 etc, so that you can nuke the directories with
        the same name without having to remove the leftovers of previous nuke.
        Also, inside the nuked-directory, the info-directory that is made to
        show who nuked it and why will now have mode 444 instead of 555.
Fix:    You now can't add a group if a private group with different case but
        same name exists.
Fix:    The site grpren command should now correctly change users' group in
        their userfile when you call it with a name that is of different case.
        Also, you can't rename a group to another group that already exists.
Change: The site ginfo command is now case-insensitive.
New:    For custom executable commands (site_cmd exec), if you specify a fourth
        argument, it will be passed to the program/script, followed by the
        user's arguments (if any). If you want to pass more than one argument,
        use [:space:] to separate them, so that they all fit into one.
Change: If a passive connection can't be established in 10 seconds, glftpd will
        now return an error and allow users to continue, instead of waiting 2
        minutes and then logging off.
Change: The kill-ghost login method (using ! as first char of username to kill
        "ghosts") will now only kill one connection (the one with longest idle
        time) instead of all connections from that user.
New:    To ban users (prevent accounts with their usernames from being created)
        add the "banned_users" option to glftpd.conf in this format:
        banned_users root billgates redhatrox bearcave
        If present, file "/rootpath/ftp-data/help/site.adduser.banned" will be
        displayed when someone attempts to add root, billgates, etc .
Fix:    "site users leech" will now correctly identify leechers in sections
        where their ratio is -1 (they have leech because -1 means disabled, so
        the ratio for section 0 counts, but site users leech didn't know that).
New:    You can now use multiple zipscripts. Simply add more post_check lines
        to your config file, with an extra argument - path mask. Glftpd will
        use the FIRST zipscript whose 2nd argument matches the directory where
        the file was uploaded. If there is no 2nd argument, or if the argument
        is "*" or "/*" or something similar, this is the main catch-all
        zipscript. To disable zipscript for only a specific path, you can use
        "none" or "disabled" as the first argument (zipscript path). Example:
        post_check none /site/public/*  # disables zipscript here
        post_check /bin/special.sh /site/special/*  # special zipscript
        post_check /bin/zipscript *     # normal zipscript for all other paths
        Note: this is useful for people who want to allow upload resume for
        certain paths (which requires disabled zipscript for incomplete files
        to be kept), but still want to test files uploaded to other places.
New:    Those who, for whatever strange reason, want to turn off glftpd's file
        download counter (stored in each file's GID) may now do so by adding
        "file_dl_count 0" to their config files.
Fix:    The default location for /etc/group file in dirlogscanner had a typo,
        so listing didn't show the correct group.
Change: The code for uploading was changed a little, hopefully to fix the
        problem with 0byte files sometimes not being deleted. Also, glftpd will
        now execute the zipscript even if the user loses connection during
        uploading, so that should allow everyone to do what they want with
        incomplete/0byte files. If zipscript is disabled, file will be left as
        it is.
Change: New users will now get current time as the "last seen" date, so they
        won't appear to have last logged in over 30 years ago.
Change: A small change that could make passive transfers faster in some cases.
New:    New cookie, %[%s]P, displays connection type if you are using the SSL
        version of glftpd.
Change: The sim_xfers setting in glftpd.conf will not affect users with the
        EXEMPT flag now. The individual limit each user has still will.
New:    New siteop command, "site errlog". It is similar to "site logins" or
        "site uchanges", but displays the error.log file. You need to add the
        line "-errlog 1" to the config file.
Fix:    Directory matching was optimized and fixed: now, it will first check if
        there is a directory exactly like what user typed but with different
        case, and if not, only then it will match it to the beginning of each
        directory. So, if you have directories "bleh" and "B", and you "cd b",
        it will go into B, instead of bleh, like it used to.
Change: The ratio cookie (which is part of the stat line) will now display the
        correct value when the creditcheck setting applies to the current
        directory, instead of displaying the user's normal ratio.
Fix:    A bug in the reset binary didn't give weekly allotment to people with
        negative credits.
Fix:    A silly bug in external nuker was resetting everyone's ratio to 3.
New:    A new cookie has been added %[%s]I+. This cookie will show the last
        time the user logged in, similar to "site seen".
New:    A new cookie, %[%d]Bo. This works similar to %[%d]B (number of users
        currently online), but it doesn't count hidden users.
New:    You can now specify if people will see colors only in the control
        connection or also in the directory listings. You can set this in the
        config file with the color_mode option.
Change: The external stats program was enhanced and fixed.
Fix:    The nuke/unnuke code (as well as external nuker) was fixed, some parts
        being completely re-written, by Bloody_A. It should fix a lot of bugs
        and stuff. Also, the third option of nukedir_style setting will now
        control the total size of the dir, not size of each file in the dir.
Change: Glftpd will only allow 2 asterisks as arguments to the LIST command.
        If you want to allow more than 2, add another argument to the lslong
        line in the config file, containing the number of asterisks you want
        to allow. 0 = unlimited.
Change: The rights section of glftpd.conf (upload, delete, makedir, etc) is
        now case sensitive, so /site/incoming will not match /site/Incoming.
Fix:    Hostnames will now be matched with no regard for the case, so if you
        are from A.com but in glftpd it's a.com, you will be able to log in.
Fix:    A minor bug when logging "site change user num_logins" to sysop.log.
Fix:    A few 'format bugs' were fixed, which in theory could be exploited.

v1.23
Fix:    A rather serious vulnerability in handling bouncers was fixed, if you
        use one, you are urged to upgrade asap. Those that don't use bouncers
        are unaffected.
Fix:    Useredit was processing credits as an unsigned number, so a negative
        number became a really big positive number.
Fix:    The use_dir_size option will now be able to count much higher than
        4 gigs if you set it to display kilobytes or gigs.
Fix:    The useredit binary was somehow compiled incorrectly - it was mixing
        ratio and credits.
Fix:    gl_spy was finally fixed to not screw up display with long filenames
Fix:    A stupid bug in site nuke and external nuker - if the nuker had zero
        bytes uploaded this month, the nukee didn't lose his/her monthly stats.
Fix:    The use_dir_size option will now work when you list files for a
        directory different than your current directory, for example if you
        were in / and did "list incoming", it was using 0 as every directory's
        size. Now, it will work the same as "cd incoming" followed by "list".
New:    You now need to specify paths for which the use_dir_size option will
        work. Just list the paths as additional parameters in glftpd.conf,
        separated by tabs/spaces. No wildcards. Every path that starts with
        what you specify will use this option (so it's recursive). To enable
        this feature for the whole site, just add /.
        Exmp: use_dir_size k /site/incoming /private
New:    A new supercookie, %LOGOUT, will cause glftpd to properly log off the
        user. This can be used in connection with some scripts to temporarily
        disable an account or something like that.
New:    The dupe_check setting in the config file takes a new parameter, "yes"
        or "no". This controls whether glftpd should ignore file case during
        dupe checking. If set to "yes", and file.zip is in the dupe database,
        glftpd will not allow File.zip to be uploaded. If set to "no", it will.
        "no" is assumed if the parameter is missing (so it's like UNIX).
Fix:    Custom site commands that use the "IS" action (which makes that command
        execute a built-in command that follows the argument) could not be
        restricted with the custom-xxx setting, only the restrictions for the
        original built-in command worked. You can now do that, so if you define
        "site_cmd last is laston", and then "custom-last 1", only siteops will
        be able to do "site last". Keep in mind that the restrictions for "site
        laston" (controlled by -info) are still in effect, so there are two
        restrictions on "site last" now, the ones following "custom-last" and
        the ones following "-info".
New:    New command line option (goes into inetd.conf), -L. It works almost
        the same as -l, the difference is it will log new directories to
        glftpd.log no matter what (-l only logs them if they're in the dirlog
        path [defined in config file]).
New:    New setting for config file, sim_xfers. It takes 2 parameters, total
        number of simultaneous downloads and uploads. -1 means unlimited; 0
        means none. Exmp: "sim_xfers 10 -1" will allow 5 simultaneous downloads
        from the site, and an unlimited number of simultaneous uploads. No one
        is exempted, so if you're trying to limit only some users, use the
        individual simultaneous transfer limit.
Fix:    External nuker had problems executing botscript for weird dirs.
Fix:    Using "timezone" disabled the feature where glftpd waits for file to
        be completely uploaded before closing download connections if someone
        is downloading the same file faster than it's being uploaded.
Change: "site ginfo" will only show full megs now instead of fractions.
Fix:    Some fixes to making directories to avoid a possible security breach.
Change: Resuming uploads should now work with more clients - glftpd will accept
        either the STOR or APPE command for this, not just APPE like before.
New:    Addition to the -noaccess setting for the config file: if the file
        ftp-data/misc/ad/command exists (command is the trigger used to control
        this command in the config file, without the leading dash), glftpd will
        display that file as the "no access" response. If it doesn't exist,
        glftpd will try to display the file following the -noaccess setting. If
        that fails, or there is no setting, it will display the default. Exmp:
        create /glftpd/ftp-data/misc/ad/who and put "No access to who" in it,
        if you want to display that error when someone has no access to "site
        who", but you want a different response for other commands.
Fix:    *sigh* another bug in site wipe - wiping files caused it to crash.
Fix:    "site give" and "site take" were sending empty messages to the person
        doing the giving/taking. I changed these messages so they look better,
        too.
Fix:    "site give" will not allow you to overflow someone else's credits
        anymore (by giving them more credits than a long integer can handle).
Fix:    Users who delete themselves won't be able to readd themselves anymore.
Fix:    "site search" will no longer show directories outside your homedir or
        privpath'ed directories. "site new" will skip privpath'ed dirs too.
New:    When a directory is deleted, botscript will now be executed with
        "DELDIR" as a parameter. Until now, it only executed with "NEWDIR",
        "NUKE" and "UNNUKE".
Fix:    "site readd user" didn't work when the user had leech, even if the
        readder's leech slots were disabled. A simple "<= 0" instead of "== 0"
        bug, oops.
Fix:    A bug in "site wipe" sometimes switched the process's uid permanently
        to regular uid, so it couldn't get back to root when it needed it -
        consequently, many file-dependent functions didn't work if directories
        in ftp-data were chmodded to 755.
Fix:    In v1.22, users with negative credits could download all they wanted.
Change: "site who" will no longer show the parameters to the PORT command
        because that's a good way to find the user's IP. "site swho" is
        unaffected.
New:    Some errors that used to go to sysop.log will now go to error.log in
        the ftp-data/logs directory. This is not viewable with site commands,
        so you might want to set up some script that checks it once in a while.
        To start the log, do "touch /glftpd/ftp-data/logs/error.log".
New:    Two new per-user settings: max simultaneous downloads and max
        simultaneous uploads. These can be changed with "site change". Setting
        these to -1 disables them; 0 prevents any uploads/downloads.
        Cookie "%[%s]I^" will display max sim uploads, and "%[%s]I*" downloads.
Change: Due to DES limitations, users will no longer be able to change their
        passwords to something > 8 characters. Before, glftpd would allow it,
        even though only the first 8 characters were used.
Fix:    If a user uploading to a "nodupecheck" directory loses connection,
        glftpd will no longer remove the file he was uploading from the dupe
        database (since the file wasn't added).
New:    The maximum number of logins for a user from the same IP is now stored
        in userfiles. It used to be available only to anonymous users through
        their password; this has been removed, anonymous passwords can only be
        @ or * now, with no number following them. This setting can be changed
        using "site change user num_logins" - it's the 2nd parameter.
        There is a new cookie for this, "%[%s]I#".
New:    The "site wipe" command will now log itself to glftpd.log.
Fix:    "site wipe -r" didn't remove subdirectories from dirlog correctly.
Change: "site ginfo" will now indicate who is a siteop or a gadmin.

v1.22
Fix:    Nukers were able to nuke the whole site if they were given nuke rights
        for it (as in nuke * *). They will not be able to nuke outside of
        their home directory now.
Fix:    The privpath config setting won't hide directories that start with
        the same name, exmp: if you privpath /site/tmp, it used to hide both
        /site/tmp and /site/tmp2.
New:    You can now specify hostnames on the bouncer_ip line - glftpd will
        try to resolve each item first.
Fix:    Gadmins who don't have any leech slots and try to readd users that
        have leech will now be denied.
Change: The built-in dupechecker will not deny upload if the file in the
        dupe database has different case than the file being uploaded. For
        example, you can now upload File.zip if file.zip already exists -
        unless your file_names rule changes File.zip to file.zip, in which
        case the upload will be denied.
Fix:    A few credit bugs concerning anonymous users and up/downloading.
        These users will not be able to reload their userfiles with any
        command now - a few commands/actions allowed that.
Change: Custom executable commands will now respond with "530 error executing
        command" if the file exits with something > 0, instead of saying
        "200 command successful" no matter what.
New:    Default home directories for new users will now be taken from the
        default.user or default.groupname file instead of the default_homedir
        setting in the config file. This way, it will work much better with
        the "site gadduser" command. If you're a gadmin, users you add will
        still inherit your own homedir regardless of what's in the group file.
Fix:    The group stats showed one more line than the number that followed
        them, ex: "site gpal 3" would show 4 lines instead of 3.
Change: If zipscript exits with 1, file will not be automatically deleted
        anymore. The error printed will still be "can't be executed".
Fix:    Site wipe didn't recurse correctly.
Fix:    Glftpd will now correctly resolve hostnames from bnc4all and dike ftp
        bouncers.
Fix:    Dirlogscanner was fixed by maestro (some rare bug)
Change: The olddirclean util was enhanced (and command line options were
        changed), the new version, 2.1, is called olddirclean2, the source is
        in /glftpd/bin/sources.
Fix:    Small fix to gl_spy; stat section fix to useredit; more fixes to
        external nuker and "site nuke".
Fix:    External nuker sometimes deleted files that were defined in ignore_type
        when nuke method was 2 (keep all files).
Fix:    Fixed more problems with glftpd, nuker and reset for sites that have
        10 stat sections.
Fix:    Fixed a problem with external nuker, where it could screw up the
        GENERAL line of userfiles.
Fix:    Using the -Tn parameter will reset default idle time to n during login
        if n is smaller than 900 (so you don't need -t).
Change: The REST command will now correctly instruct users that they need to
        issue "APPEND" or RETRIEVE, not "STORE" or RETRIEVE.
Change: Glftpd will no longer delete 0-byte files if the upload was
        completed successfully - only when connection wasn't established.
        You can easily delete 0byte files through zipscript - I added the
        code to the base zipscript if you want to see how it's done.
Fix:    Small bug in grpren which allowed spaces to be at the beginning of
        the new group name.
New:    [:hash:], if found in config file, will be converted to #.
New:    The stats binary was updated by HoE to show daytop and monthtop for
        groups.
Change: Nukers are no longer allowed to delete files in the directories where
        they can nuke. If you want to give them delete access to some,
        directories, add the A flag to the delete lines in your config file.
Change: Rewritten "SITE DUPE" command, check the site.dupe file in ftp-data/help
Fix:    Fixed small bug in MDTM output
Fix:    Ok I finally fixed the bug with bandwidth control, where it wouldn't
        work if the limit was below 8K/s. It should work down to 1K/s now,
        possibly even lower, and should be more accurate.
Fix:    Fixed the bug in lastonline code that would leave lock files and ftp
        ghosts if the lastonline file didn't exist.
Change: Some cosmetic changes to user interface when changing flags.
Fix:    If a user's tagline contained "%s" (and probably other output format
        codes too), glftpd would crash during "site who". It's still a bad idea
        to have those in taglines - glftpd still won't allow users to use them.
New:    If a user's first IP starts with '!', glftpd will try to read a list of
        IPs from a file that follows the !. One IP per line, make sure there
        are no carriage returns at the end of each line (just a linefeed). It
        has to be the first IP. You have to add it manually to the userfile,
        it can't be added with a site command (for security reasons). If an IP
        in that file starts with a !, it will be banned, meaning this user will
        not be allowed to log in if his ident@ip matches. If this file does not
        exist or the user's IP is not on the list, glftpd will continue
        checking the rest of IPs in the userfile the old way.
Change: The MKD command will now respond with the name of the directory created.
Fix:    2 stupid bugs with the idle time: the idler flag stopped working, and
        issuing an unrecognized command caused your idle time to be set to
        60 seconds. Oops. Thanks to Zio for debugging it with me.
New:    New users will now get a default idle timeout of 60 seconds until they
        log in successfully, at which time it will become whatever the -t
        parameter in inetd.conf is. This way no one can easily block all your
        slots by connecting and not logging in.
Fix:    There was a bug in the lastonline code - if you defined the number of
        lines to be more than 50, it would corrupt the file. Now, any number
        bigger than 50 will be treated as 50.
Fix:    When doing "cd 1", if newest directory in dirlog was outside this
        user's homedir, he/she would get an error message containing the
        newest directory. Glftpd will now just say "Invalid Directory".
Fix:    In some rare cases, users were allowed to idle forever without having
        the idle flag. I believe this happened sometimes when establishing
        a data connection failed. It should be ok now - at least it stopped
        happening to me after this fix.
New:    Since so many people are too lazy to delete old stuff from the shell
        or make an external script to do it via ftp (DELE removes credits and
        stats from the uploader), I coded a "site wipe" command.  If the
        argument is a file, it will simply be deleted. If it's a directory,
        it and the files it contains will be deleted.  If the directory
        contains other directories, the deletion will be aborted.  To remove
        a directory containing subdirectories, you need to use "site wipe -r
        dirname". BE CAREFUL WHO YOU GIVE ACCESS TO THIS COMMAND. Glftpd will
        check if the parent directory of the file/directory you're trying to
        delete is writable by its owner. If not, wipe will not execute, so
        to protect directories from being wiped, make them 555. Also, wipe will
        only work where you have the right to delete (in glftpd.conf). Delete
        right and parent directory's mode of 755/777/etc will cause glftpd to
        SWITCH TO ROOT UID and wipe the file/directory.  "site wipe -r /" will
        not work, but "site wipe -r /incoming" WILL, SO BE CAREFUL. To give
	access to site wipe, add "-wipe -user or =group or flag" to config.
New:    Added 5 new custom flags - J, K, L, M, and N.  You can use these in
        the config file to give some users access to certain things without
        having to use private groups.  These flags will only show up in
        "site flags" if they're turned on.
Fix:    Fixed a bug that I created in 1.21 - uploading 0byte files was causing
        glftpd to 'hang' - oops ;)
Fix:    The nuker binary (external nuker) was corrupting dirlog because of an
        outdated record definition (one symptom was that nuked directories did
        not disappear from 'site new').
Change: If you're using the * password for anonymous users, the actual password
        used will be logged to login.log, just like with the @ password.
Change: Looks like the LOGOUT problem in glftpd.log wasn't fixed right.  I
        removed 2 instances of glftpd adding the LOGOUT entry before the LOGIN
        entry was added. Hope this fixes it.
Fix:    Fixed a bug that made the process's name 3-4 lines long, with spaces
        following the hostname. Now, NULL will follow hostname. As a result,
        entries in your syslog (if you have -d enabled in inetd.conf) will
        look the way they should have.
Change: The variables storing total bytes for show_totals have been changed
        from signed to unsigned, so they can store twice as many bytes now,
        and they won't become negative.
Fix:    "site take" had the same bug "site give" had (site take name amount
        msg didn't send the msg).
Fix:    Making directories in a certain way by-passed path-filter and created
        the dir even though it should have been denied. Same with uploading
        files.

v1.21
Change: The "file_names" and "dir_names" options for the config file changed.
        The new format is: dir_names 1/0 [lower/upper] [XY] [XY] ...
        The first parameter is the same: 1 means capitalize first letter.
        If the second parameter is anything but lower or upper, it will be
        ignored.  The third parameter is a list of character conversion pairs.
        Character X will be translated to character Y. If Y is the word "NULL"
        then character X will just be deleted.  Example:
        dir_names 0 bleh [( ]) [:space:]_ ,NULL
        This will convert all square brackets to parentheses, spaces to
        underscores, and delete all commas.  Same goes for file_names.
New:    All site commands can be re-wired to custom site commands now.  For
        example, in order to invoke the "site who" command when someone types
        "site usersonline", add "site_cmd who is usersonline" to the config
        file. "Is" is the parameter that causes the re-wiring.
Change: The hideuser option in the config file will now take multiple
        parameters in the '=group -username flag' format.  You need to put all
        your hideuser lines together on one line, since only the first occurrence
        will be looked at.
Fix:    (Probably) fixed the bug of corrupting userfiles and/or crashing when
        you add more than 10 sections to the userfile.
Fix:    ls -lR will now strip the home directory from every directory whose
        contents it lists. ls -ld will also strip the home directory.
Fix:    Doing "site user blah" resulted in an error message in the sysop.log
        file if blah was not a valid user.
Change: The cookie files for "site nukes" have been re-designed.  Each nuke
        entry will take up 2 lines.  You don't have to use these.
Change: Fixed a problem with stat sections.  If you specified the path as
        /site/section/*, going into /site/section didn't switch you to that
        section, you had to go into the subdirectories. Another solution was
        to use /site/section* as path, but that wasn't a good fix.
Change: The hideinwho option will now hide the user regardless of what he/she
        is doing, not just when he/she is transferring files.
Fix:    The nuke right will now behave like other rights; if you make it
        /site/incoming/*/, this will only apply to dirs under incoming, but
        not to their directories.
Change: The function responsible for adding total size of all files in a dir
        (for use with the use_dir_size setting) was improved and should be
        a little faster now.
Fix:    Useredit was fixed to not show ?????? in the bytes fields when showing
        user stats (when you press ->).
Fix:    Another bug in site nuke AND external nuker, empty directories were
        not causing any credit penalty in some cases.
Change: It looks like scripts can return 127 instead of 1 in some cases when
        they can't be executed, so exit code 127 will be treated just like 1
        by glftpd.
New:    The "pasv_ports X Y" setting, if present in the config file, will cause
        glftpd to select a port between X and Y when making a passive
        connection.  Without this, the next sequential port is used.  You
        should have around 10 ports per user, just to be safe, so if your
        max_users is 30, make the range at least 300.
Change: UNFO in userfiles is now called TAGLINE. Glftpd will auto-convert.
New:    Userfiles will now carry a new line, DIR, whose parameter will be the
        user's starting directory.  Do not include the rootpath or the homedir
        in this. exmp: DIR /incoming will have the user start in
        /glftpd/site/incoming.
Change: Since I upgraded my system, glftpd will now be compiled on libc6 (also
        known as glibc2). Also, it will be a static binary, so that variations
        in libc versions won't cause different behavior on different boxes.
Fix:    Looks like I broke the calc_crc feature in 1.20.
New:    The DIR option in userfiles can be changed through ftp with "site 
        change <user> startup_dir <directory>".
Fix:    Someone found a bug in site give that was there probably the whole time
        (I don't remember ever messing with site give).  Anyhow, site give
        username 50000 blah blah blah will now send the "blah blah blah"
        message to username, like the docs say it's supposed to.
Fix:    2 more nuke bugs: glftpd was dividing by 1048676 instead of 1048576 to
        get megs from bytes, and site unnuke was subtracting more from the
        bytes_nuked variable than it should, resulting in incorrect nuketop.
Fix:    A small bug with site kick (adding stupid errors to sysop.log).
Fix:    If a user was connected but didn't log in yet and someone did a site
        who, glftpd would add an error to sysop.log saying userfile for
        -NEW- could not be opened.
Fix:    Thanks to Zio, who found these, 2 bugs were fixed, with site nuke and
        with the dele command, that caused weird lock files to be created in
        the ftp-data/users dir.
New:    The "lastonline" option in glftpd.conf can now take an optional 3rd
        argument.  If that argument is 1, users who didn't upload, download,
        nuke or add a user and who timed out will not be logged.  If it is
        2, these users won't be logged even if they lost connection or quit
        normally, not just when they timed out.  This only affects the
        "site laston" list.
Fix:    Lockfiles will no longer appear in "site users" or "site user".
New:    Oneliners now show the date before the name.  As a result, they are
        limited to 57 characters instead of 60, or they wouldn't fit on
        regular 80-column screens.
Fix:    Fixed a bug with "site grpnfo" that would add a new group if you used
        it on an existing group with different capitalization.
Change: Glftpd will now delete 0-byte files created by the STOR command if the
        preceding PORT/PASV command was unsuccessful.  As a side effect, you
        won't be able to leave messages using "quote stor your_msg_here"
        anymore, but I think it's worth the trade-off since it's easy to make
        a custom command for leaving messages.
Change: The pasv_addr setting was causing glftpd to tell the client which IP
        to connect to, but it was really binding to the default interface.  I
        changed it to bind to that IP instead.  If you need to work the way it
        used to, add another parameter to pasv_addr, 1. ex: pasv_addr 24.1.2.3 1
Fix:    Deleting files smaller than 50k resulted in corrupted stats in
        userfiles.
Fix:    The nuke and unnuke commands (and external nuker) will no longer report
        that a user with leech lost credits when nuking more than 1x (the user
        didn't really lose credits, but glftpd was reporting that he/she did).
Fix:    Users trying to log in with a bad ident@ip were causing glftpd to add
        a LOGOUT entry to glftpd.log, which shouldn't be there.
Change: The privpath setting didn't allow you to cd to the directory specified
        as first parameter, but it let you cd to subdirectories of that dir.
        It won't any more.
New:    Added code to prevent making directories or uploading files inside
        private dirs to which the user doesn't have access.

v1.20
Change: "site gadduser" will now use default.group, if it exists, instead
        of using default.user, so it will work the same as if a group admin
        was adding this user (except flag 3 will not be automatically assigned
        and the user will not inherit the adder's home directory).
Change: When a nuked directory contains other directories, they will be
        chmodded to 555 so that no one will be able to upload to them.  Unnuke
        will chmod all subdirectories of the unnuked directory back to 777.
New:    Added code that will allow siteops to set up UNLIMITED custom scripts
        to run before/after any command they choose.  To do this, you need to
        add the script in the following format to the config file:
	cscript <command> <when> <path/filename>
	If using a command with a space in it, like a site command, you need
        to use "[:space:]" between the words, ex: SITE[:space:]WHO.
        <when> can be either "pre" (to execute this script before executing
        this command), or "post" (to execute it after).  Path/filename is the
        full path to the script, ex: "/bin/sitewhoscript.sh".
        "post" scripts won't be able to echo anything because they are executed
        after glftpd sends the response to the client - can't do this any other
        way. If a "pre" script exits with anything bigger than 0 (or if it
        can't be executed), glftpd will NOT execute the command which should
        run after the script.  Also, it is YOUR responsibility to echo the
        correct MESSAGE CODE with every echo line that you use - if you echo
        the wrong code, ftp clients may 'hang'.  To find out what code you need
        to use, just look at what glftpd uses when that command is executed.
        Example: if you run a script before "site who", and you want to print
        something from it, use: echo -e "200Your message here\r".
        If you exit a pre script with 1 in order to prevent the user from
        executing the command, you might want to use a generic error code with
        your echo lines, like 500, which will tell the clients that this
        command was denied - or use the error code that glftpd uses when
        denying execution of this command.
        Both kinds of scripts will be passed 3 parameters:
        $1 = full command string the user used, $2 = user's login name,
        $3 = user's primary non-private group.
New:    Online logs (uchanges, logins, and reqlog) can now take a second
        parameter, a string to search for.  "site logins 0 lamer" will
        display all entries with the word "lamer" in them;  "site uchanges 10
        purge" will display all purges from the last 10 days.
New:    If zipscript returns a number between 10 and 1010, glftpd, after
        deleting and unduping the file, will sleep for (the number returned
        minus 10) seconds. So, if you want to prevent people from uploading
        incomplete files in rapid succession, just change your exit code to 10
        more than the number of secs you want glftpd to wait. ex: "exit 25"
        will make glftpd sleep for 15 secs.
New:    New cookie, %[%s]b, will print the name of current section.
Change: The statline.txt cookie file can now contain multiple lines; all the
        lines will be displayed after directory listing, transfer, or doing
        "site stat".
New:    A new environment variable, SPEED, will be exported after every
        upload/download, so it can be used in zipscript or a post-script
        running after the RETR command - it will contain the kilobytes/sec. ex:
        echo -e "$SPEED\r" will print 500 if you transferred the file at 500K/s.
Fix:    If downloading at really fast speeds (1000k/s and up), glftpd would
        report a much lower speed because it would write to logs first and
        calculate speed after that, instead of right after the transfer.
Change: The lines in etc/group can now be twice as long as before.
Change: Nuke/unnuke reason was increased from 30 to 60 characters.  Since
        nukelog is a binary file, you have to start a new nukelog (unless you
        write a converter).  Also, to actually use this, you need to edit your
        cookie file and increase the reason field's size.
Change: The secure-ip code was cleaned up.  Among other things, it will no
        longer let you add a hostname with no ident at all, you need to use
        *@hostname, so you'll be under the rules of the no_ident setting.
Change: The ratio cookie will show "Leech" instead of "Unlimited" when the
        user's ratio is 0.
New:    2 new cookies for "site who" and "site swho": %[%s]Wx, for total upload
        speed by all visible users, and %[%s]Wy, for total download speed.
        Probably the best place to use these is in the who.foot file.
Fix:    Looks like anonymous users kept generating a lockfile every time they
        logged in - how could it be that no one noticed this??
New:    A new right, "deleteown", for the rights section in the config file,
        which gives users permission to delete their own files (their stats and
        credits will be adjusted).  To have glftpd work like in previous
        versions, you need to add "deleteown * !8 *" to the config file.
        If a user has rights to delete files via the delete/nuke rights, the
        deleteown right will have no effect.
Change: "site ginfo" will now show "***DELETED***" instead of a user's tagline
        if he/she has the delete flag enabled.
Change: The "site readd" command is now gadmin-restricted, meaning that if
        gadmins have access to it via the config file, they will only see
        deleted users in their own group.  "site readd username" already was
        restricted, so no changes to that.
Change: The "site purge" command is now gadmin-restricted, so if you give
        gadmins access to it, they will only be able to purge users from their
        own group.
Change: Users will only be able to see new directories created under their
        homedir's tree with site new, so a user with homedir /site/lamers will
        not see directories created in /site/incoming (but you can still use
        the %NEW cookie, with /site/incoming as a parameter, in a msgpath file
        or as a custom command to display it for them).
New:    gl_spy and useredit were enhanced and a few small bugs were fixed.
Fix:    A bug in "site ginfo" was putting error messages into sysop.log.
Fix:    Fixed a bug that made the speed outrageously high if a user was
        downloading a file for more than 3 hours.
Fix:    When a user restarted a download, he was losing credits for the whole
        thing, but his stats only reflected the resumed part. Not anymore.
Change: The show_diz cookie files are now called show_totals.
Fix:    Numerous fixes to the rename/rmdir functions.
Change: Dupelog is back to what it used to be - new entries are written at the
        end.
New:    Site dupe will now only list the specified number (or 20 if no number)
        entries from the bottom of the file, instead of from the top.  It will
        also take multiple words, ex. "site dupe linux free office" will find
        entries like "Free.linux.office.stuff" but not "linux.office.stuff"
        (in other words, AND is assumed between search terms). This was done
        by _HoE_.
New:    I created site.help.kick and added it to the site.help file to be
        displayed to people with flags D or E.  It contains 3 commands:
        kick, kill and swho.
Fix:    Fixed a y2k bug in the mdtm command :).
New:    A new config file setting, secure_ip. This will let you define who
        can add what kind of IPs to users. You can specify who can add
        non-numeric hostmasks (as in a@*.aol.com) and how many fields must be
        fully-numeric in numeric IPs (without wildcards). The first parameter
        is the minimum number of numeric fields; the second should be 1 if
        these people are allowed to add non-numeric hostmasks, 0 if not. The
        rest of parameters are permissions, as with secure_pass. ex:
        "secure_ip 1 1 =STAFF 1" means staff and siteops only need one numeric
        field in a numeric IP and they are allowed to add non-numeric hostmasks.
        "secure_ip 3 0 *" means that everyone else needs at least 3 numeric
        fields in a numeric IP and can not add non-numeric hostmasks.
	You can have several secure_ip lines. First match wins, so put the
        most specific on top and a catch-all case at the end.

v1.19
Fix:    SITE NUKE was crashing for 1% of users, it should work for everyone
        now.
Change: The max value for num_logins was increased from 9 to 30000.
Change: If an upload was around 3 gigs, glftpd would list the dir's size as
        a negative number.  I changed this 'long' to be unsigned, so it will
        now handle directories twice as big as before.
Fix:    A nasty bug would overwrite your username if you changed your tagline
        to something longer than 63 characters.  The only problems that I know
        of that resulted from this are a blank slot in site who and the ability
        to log in more than what your num_logins was set to.
Change: The configurable site commands (at the bottom of the config file) are
        more configurable now.  You can use -username to give access to a
        specific user.  ! can also be used in front of groups or users, not
        just in front of flags.
Change: grppath is now called privpath.  Also, it will now take flags, =groups,
        or -usernames instead of just group names. NOTE: Wildcards are still
        not allowed in the path setting - it would slow down listing directories
        too much.
Change: The rights section went through a similar change.  First of all, remove
        the word "yes" and "no" from those lines.  Second, before every group
        name add a =.  You can also add flags or -usernames.  If you had a line
        with a "no", just add a ! in front of every group/person/flag you wish
        to exclude from this right and add a "*" as the last argument.  The
        affected rights are: upload, delete, nuke, overwrite, dirlog, makedir.
Fix:    "site users deleted" didn't work.
Change: Glftpd will now write a new directory to the beginning of dupelog instead
        of appending it at the end, so the results of "site dupe" will be
        newest-first.  Note that this requires re-writing the dupelog file each
        time someone makes a directory.  If your dupelog is big, you might need
        to cut it down if "mkdir" is slow.  If I get reports that this results
        in a significant slowdown, I might go back to the way it was before and
        just write a util to run once a day that will sort the dupelog.
Change: The syntax of "secure_pass" changed.  The groups/users/flags on the
        right now mean users that this rule applies to instead of users who
        are exempted from this rule.  Also, you can now have multiple rules,
        for example if you want a different rule for siteops and a different
        one for normal users, or a different one for several group.  The first
        match with a flag/group/username stops the search, so make sure to put
        the tightest rules on top.  See examples in config file.
Change: The "no_downloads" setting is obsolete.  Instead, use "download" in the
        rights section.  Ex: "download * *" will allow everyone to download
        everything.  If you have "download /private/* 1" above that, only
        siteops will be able to download from the /private directory.
Change: The "freedldir" setting changed name and format.  It is now called
       "freedl" and  looks like any other right from the "rights" section,
        meaning it can take multiple groups, usernames, or flags.
Change: The "freefile" setting went through the same change.
Change: "nostatdir" is now "nostats" and the rules changed same as above.
Fix:    There was a serious bug in unnuke.  If a dir was nuked at other than 1x,
        then the nukee would get back no stats back (if 0x) or # times size of
        stats back (#x) instead of 1 times size.  It's so sad no one found this
        until I *accidentally* noticed it during other testing.
Change: "hidedir" is now called "hideinwho".  Yes I know it sounds funny, but it's
        much more descriptive not, so no one will confuse it with privdir (former
        grppath).  Also, siteops are no longer exempted by default.  If you want
        siteops to see people who are in private dirs, change the "*" at the end
        to "!1 *".
Change: "show_diz" will no longer display directory totals (also known as race
        info).  To get that, enable show_totals.  The syntax is the following:
        "show_totals /site/incoming/*/".  This will show totals for any directory
        in /site/incoming.  You can have several lines of show_totals.
New:    You can now have different welcome messages for different users.  Just
        add a new "welcome" line and flags/groups/users to which this message
        should be displayed, i.e. "welcome /ftp-data/welcome2.msg =STAFF".
New:    Same with "goodbye".
New:    Similar story with the "newsfile" line, with one difference.  If a match
        is found, glftpd keeps searching for and displaying other newsfile lines.
        With welcome and goodbye, the search stops on first match.
Change: The no_ident rule's effect has been reversed.  Instead of the groups/
        users/flags on the right specifying users exempted from this rule,
        they now specify users to whom this rule applies.
Change: "msgpath" can now also take flags/groups/users.  Glftpd will display
        every file on a msgpath line if both the path (first argument) matches
        and the user has a flag/is in a group listed on the right.  You need
        a trailing slash for the path if you don't use wildcards.
Fix:    Another fix in site nuke - it would crash if you tried to nuke files
        by a user that was not in the passwd file.
New:    The stat commands (wkup, aldn, etc) can now take a section name or
        number as the 2nd parameter (the 1st parameter must be the number
        of lines to display).  The stats cookies (%WEEKUP etc) will also
        work like this.  If the second parameter is missing, stats will be
        shown for the current section (just like before).
New:    A new cookie, %[%s]Id, will print a 'Y' if the user has messages
        waiting and an 'N' if not.
New:    The "site group" and "site chgrp" commands are now case-insensitive.
        Instead of having to do "site chgrp bleh StAfF" (assuming you're into
        that kind of names) you can do "site chgrp bleh staff" and group
        "StAfF" will be added to the user's group list.
Change: The "calc_crc" setting will now take multiple arguments, which can
        be either file masks (*.rar) or directory masks (/site/Rars/*).
        Only files that match one of the parameters to calc_crc will have
        their CRC code calculated and passed to zipscript.  NOTE: For dirs,
        if you don't have a wildcard at the end, you need the trailing '/'.
Fix:    LOTS of bugs fixed in external nuker, and a few small ones in internal
        nuke/unnuke routines.
Change: Killing of ghosts (by logging in with ! in front of username) will now
        be logged into login.log instead of sysop.log.
Change: The "tagline" option in the config file wasn't working like the docs
        implied for a long time.  The default tagline is what's in the
        default.user file, the only thing this was used for was to enforce
        taglines.  So, you don't need a 1 or a 0 at the end anymore; if this
        option is present, and if the user's tagline is the same as argument #1,
        he will be forced to change his tagline.
Fix:    Fixed a small problem in ftpwho and gl_spy that would say that a user
        is still uploading (at 0k/s) after he/she finished.
Fix:    Fixed a rather serious bug of people being able to download without being
        seen if using ASCII mode.  Most files will be corrupted when downloading
        this way, but some (gzips) will not.

v1.18
Change: As you can see, version numbers changed from 1.xx.xx to 1.xx. Version
        1.17.3 was a private beta version of 1.18, so 1.18 is the next
        public version after 1.17.2.
Fix:    I forgot to delete part of the code I re-wrote, so nukers were unable
        to delete files even if they had delete access.  Once again, thanks
        to all the beta testers who never reported this.  Why am I the only
        one who finds all the bugs???
Fix:    Site update was fixed to inform you why it wouldn't work in some cases.
Fix:    glupdate.c was changed to NOT strip rootpath from the directories if
        it was "/", so that directories will get added as "/glftpd/site/bleh"
        instead of "glftpd/site/bleh".
New:	site update and glupdate.c are now fully recursive.  That means that
        when you have subdirectories in the directory you're updating, their
        size will be counted and added to the main directory's size. For example,
        directories with CD1 and CD2 inside will no longer show up as empty.
Change: glupdate once again.  It will now take the -r switch to specify a
        non-standard config file, so you don't have to recompile it. A bug was
        also fixed which made glupdate corrupt dirlog if you compiled it on a
        libc6 system.
Change: Directories with more than a gig of files inside will now be shown in
        site new with a size of x.xxxG instead of xxxx.xM, which used to cause
        this field to run into the number of files field.
New:    After executing a custom command, glftpd will now reload that user's
        userfile in case the external process made changes to it.
Fix:    glftpd will now successfully compile and work on libc6 systems (but
        we will still be compiling new binaries on libc5).
New:    You can now make glftpd show directories' size (the total size of all
        files in that directory), instead of the number of bytes the directory
        takes up, when a user does a "LIST".  It will look like this:
        drwxrwxrwx   1 user  group      dddd Nov 22 16:49 incoming
        (dddd is the number of kilobytes the files in that directory occupy).
        To enable this, add "use_dir_size k" to the config file. You can also
        use b or m instead of k, changing the display to be in bytes or megs.
New:    Reset will now take a switch (-a) that lets you reset alltime stats.
        I still think that's lame but I'm sick of people asking for it.
Fix:    SITE GADDUSER was screwing up unix clients if the user you tried to
        add already existed.
New:    If there was only 1 meg of free space and someone was uploading files
        3 megs in size, all of them would fail, resulting in a big waste of
        bandwidth. You can now add "free_space x", x being a number (in megs),
        to the config file, which will prevent people from uploading when the
        current directory's free space is less than x.
Fix:	2 minor fixes made for the timeframe feature.
Fix:    Directory matching expanded to match directories at the end of path,
        like /site/incoming/bl, which will now match "/site/incoming/bleh".
        This fixes problems with flashfxp if you drag a directory whose name is
        converted by glftpd (to upper/lowercase, etc).
New:    SITE NEW will now accept "." as a parameter, and will only show new
        directories in the directory tree you're in instead of the global list.
        If you still want to specify the number of matches to display, you can
        put it after the ".", i.e. "site new . 5".
        EXMP: "site new ." will show 10 newest dirs in current directory tree.
Fix:    Gadmins were able to cheat and add users past their allocated slots by
        deleting a user, adding a new one, then readding the deleted one with
        site readd (well it's in the open now so upgrade quickly :}).
Fix:    The code that shows only x number days of a log file (for example:
        site uchanges 2) was in a very bad shape, I fixed many problems so now
        it should work fine (never code while you're drunk, eh?).
Fix:    When nuking someone who had leech, glftpd was leaving the userfile
        locked so there were some warning messages in the sysop log.
Change: The SITE USERS command was re-designed, it should be a little faster
        now and it won't list all users if you use the wrong option.
Change: SITE GROUPS will no longer count deleted users.
New:    If a user's password is "*", any password will fit. If it is "@", only
        passwords that look like email addresses will fit. Email passwords will
        be logged to the login.log. You can specify a single number after '@'
        or '*' which says how many connections from the same IP are allowed for
        this user, so "site chpass anonymous @2" will allow email for password
        and will not let the user log in if he is already logged in twice from
        this IP.
        You have to use "site chpass" to change a user's password to * or @.
Change: SITE UNFO is now SITE TAGLINE.  Actually, both will work right now,
        but UNFO will probably be removed in the future, so start using TAGLINE.
Fix:    The total bytes at the bottom of "site users" was overflowing at ~2
        terrabytes.  It should store over 4 thousands terrabytes now :}.
Change: Userfiles were storing bytes up to ~2 terrabytes, they will store ~4
        terrabytes now (changed from signed to unsigned long).
Change: When nuking more than 1x, both stats and credits were decreased by more
        than 1x. Now only credits will decrease more, stats will only be decreased
        by the exact size of what's nuked.
New:    Weekly allotment now supports sections. The syntax for changing it now is:
        site change user wkly_allotment "#,###", where the first number is the
        section number (0=default section). Only one section at a time is supported,
        if you want it for more than one you will need to write a script and run it
        from crontab to add the credits to the user.
Change: Change all your "exit 1"'s in dupescript, zipscript and dirscript to
        "exit 2".  Glftpd will now respond with a built-in error message when
        one of these scripts returns a 1, so that it will be easy to tell when
        these scripts can't be executed.
New:    Many more site commands are flag-configurable now, see file "newconfig.118"
        for details.  Also, all the help-commands ("site blah" if blah requires a
        parameter) will only work if you have access to the command.
New:    You can define your own "You do not have access to this command" now.
        Just add "-noaccess /ftp-data/text/bleh.txt" to the config file and create
        a file displaying your message.  You can use cookies in that file.
New:    On-the-fly CRC calculation.  If you add "calc_crc 1" to the config file,
        glftpd will calculate the CRC of files being uploaded and pass this
        string to zipscript.  You can then call the flysfv binary from zipscript
        to compare that string with what it finds in the sfv file (see the default
        zipscript for an example).  This procedure will save some time if several
        people are uploading files at the same time and you have a slow harddrive,
        but it might slow the upload down a little, so use it at your own risk.
        Thanks Hoopy for this feature.
Fix:    grppath was matching all directories that started with the same name. It
        will only match directories that have the exact name now.
Fix:    cd ~ was causing problems.  ~ will be treated as / from now on, since
        everyone's apparent homedir is /.
Fix:    A rather serious bug that would allow someone to execute commands on your
        box was fixed.  This exploit could also be used to start some kind of a
        daemon on your box that would allow people to connect to it and log in.
        Make sure you enable your path-filter in the config file to close this
        security hole.  See the UPGRADING file for an example.
New:    SITE PASSWD can now disallow insecure passwords.  All you need to do is
        add "secure_pass xxx" to the config file. Glftpd will not allow passwords
        that are shorter in length than xxx. Also, you can specify the minimum
        number of capital and lowercase letters, numbers, and other symbols.
        Example: "secure_pass Ab1..." will only allow passwords 6 characters or
        longer that contain at least one capital letter, one lowercase letter,
        and one number. The '.' is a placeholder. You can use any letter, number,
        or punctuation character (except '.') in this string.
	Other arguments (optional) are flags/groups that are exempted from this
        rule when using the site chpass command or adding users.
