[![Wyeomyia smithii](https://github.com/okeuday/pest/raw/master/images/320px-Wyeomyia_smithii.jpg)](https://en.wikipedia.org/wiki/Mosquito#Lifecycle)

Primitive Erlang Security Tool (PEST)
-------------------------------------

[![Build Status](https://secure.travis-ci.org/okeuday/pest.png?branch=master)](http://travis-ci.org/okeuday/pest)
[![hex.pm version](https://img.shields.io/hexpm/v/pest.svg)](https://hex.pm/packages/pest)

Do a basic scan of Erlang source code and report any function calls that may
cause Erlang source code to be insecure.

The tool is provided in the form of an escript (an Erlang script) which may
also be used as a module.  Usage of the script is provided with the `-h`
command line argument, with the output shown below:

    Usage pest.erl [OPTION] [FILES] [DIRECTORIES]
    
      -b              Only process beam files recursively
      -c              Perform internal consistency checks
      -d DEPENDENCY   Expand the checks to include a dependency
                      (provide the dependency as a file path or directory)
      -D IDENTIFIER   Expand the checks to include a dependency from an identifier
      -e              Only process source files recursively
      -h              List available command line flags
      -i              Display checks information after expanding dependencies
      -m APPLICATION  Display a list of modules in an Erlang/OTP application
      -r              Recursively search directories
      -s SEVERITY     Set the minimum severity to use when reporting problems
                      (default is 50)
      -U COMPONENT    Update local data related to a component
                      (valid components are: crypto, pest/dependency/IDENTIFIER)
      -v              Verbose output (set the minimum severity to 0)
      -V [COMPONENT]  Print version information
                      (valid components are: pest, crypto)

Erlang/OTP version 19.0 and higher is required.
If beam files are used, they must have been compiled with the `debug_info`
option to provide the `abstract_code` used by pest.erl.  However, pest.erl
also consumes Erlang source code, including Erlang source escript files.
If beam files are available, it is best to use the beam files with pest.erl
due to how the Erlang compiler preprocessor and optimizations can influence
function calls.

Please feel free to contribute!  To add security problems to the scan
insert information into the [list of checks](https://github.com/okeuday/pest/blob/master/src/pest.erl#L153-L215).

Usage
-----

To scan any `.beam` files in a `lib` directory recursively, use:

    ./pest.erl -b /path_to_somewhere/lib

If you want to see all possible checks,
just turn on the verbose output with `-v`:

    ./pest.erl -v -b /path_to_somewhere/lib

To check version information related to Erlang/OTP crypto, use:

    ./pest.erl -V crypto

To do a slower scan that includes indirect function calls from Erlang/OTP
(as described in [Indirect Security Concerns](https://github.com/okeuday/pest/#indirect-security-concerns)), use:

    ./pest.erl -v -b -d /erlang_install_prefix/lib/erlang/lib/ /path_to_somewhere/lib

Determining checks that include all indirect function calls for Erlang/OTP 19.3
can take several minutes, so it is easier to use cached results.
The checks have already been cached for Erlang/OTP 19.3 (with the command `./pest.erl -v -b -d /erlang_install_prefix/lib/erlang/lib/ -U pest/dependency/ErlangOTP/19.3`),
which can be used to obtain the same output with:

    ./pest.erl -v -b -D ErlangOTP/19.3 /path_to_somewhere/lib

Test
----

To have pest.erl check itself, use:

    ./pest.erl -v -c -D ErlangOTP/19.3 ./pest.erl

Indirect Security Concerns
--------------------------

Usage of various Erlang/OTP dependencies can have their own security concerns
which Erlang source code may depend on indirectly.  To provide a representation
of security concerns related to Erlang/OTP dependencies, the pest.erl script
was ran on all of the beam files installed for Erlang/OTP 19.1, with the result
provided below:

    $ ./pest.erl -v -b ~/installed/lib/erlang/lib/
     90: Port Drivers may cause undefined behavior
         erl_ddll.beam:153 (erl_ddll:try_load/3)
         megaco_flex_scanner.beam:89 (erl_ddll:load_driver/2)
         dbg.beam:[436,443,474,481] (erl_ddll:load_driver/2)
         wxe_master.beam:126 (erl_ddll:load_driver/2)
     90: NIFs may cause undefined behavior
         asn1rt_nif.beam:[57,69] (erlang:load_nif/2)
         crypto.beam:[645,658] (erlang:load_nif/2)
         erl_tracer.beam:36 (erlang:load_nif/2)
         dyntrace.beam:[84,96] (erlang:load_nif/2)
     80: OS shell usage may require input validation
         ct_webtool.beam:[172,175,189,194,198] (os:cmd/1)
         test_server_node.beam:640 (os:cmd/1)
         dialyzer_callgraph.beam:757 (os:cmd/1)
         icpreproc.beam:55 (os:cmd/1)
         observer_wx.beam:537 (os:cmd/1)
         cpu_sup.beam:246 (os:cmd/1)
         memsup.beam:[685,698,729,772] (os:cmd/1)
         os_sup.beam:242 (os:cmd/1)
         yecc.beam:457 (os:cmd/1)
         system_information.beam:[571,572] (os:cmd/1)
         snmp_config.beam:[1560,1567] (os:cmd/1)
     80: OS process creation may require input validation
         prim_file.beam:1056 (erlang:open_port/2)
         prim_inet.beam:87 (erlang:open_port/2)
         ram_file.beam:400 (erlang:open_port/2)
         megaco_flex_scanner.beam:114 (erlang:open_port/2)
         os_mon.beam:[89,95] (erlang:open_port/2)
     15: Keep OpenSSL updated for crypto module use (run with "-V crypto")
         ct_config.beam:636 (crypto:block_decrypt/4)
         ct_config.beam:602 (crypto:block_encrypt/4)
         ct_make.beam:285 (compile:file/2)
         ct_netconfc.beam:[1920,1933,1939,1953] (ssh:_/_)
         ct_netconfc.beam:[1141,1924,1926,1947] (ssh_connection:_/_)
         ct_slave.beam:426 (ssh:_/_)
         ct_slave.beam:[427,429,433] (ssh_connection:_/_)
         ct_snmp.beam:[546,578,750,766,781,797,812,827,842,858] (snmp_config:_/_)
         ct_snmp.beam:[506,516] (snmpa:_/_)
         ct_snmp.beam:[264,277,294,295,634,658,666,671,677,682] (snmpm:_/_)
         ct_ssh.beam:[966,969,1236] (ssh:_/_)
         ct_ssh.beam:[996,1001,1009,1024,1027,1046,1054,1067,1256,1261] (ssh_connection:_/_)
         ct_ssh.beam:[971,990,1076,1082,1088,1094,1100,1106,1112,1118,1124,1130,1136,1142,1148,1154,1160,1166,1172,1178,1184,1190,1196,1202,1208,1214,1220,1240] (ssh_sftp:_/_)
         compile.beam:1337 (crypto:block_encrypt/4)
         CosFileTransfer_FileTransferSession_impl.beam:[129,625,703,707,857,859] (ssl:_/_)
         dialyzer_cl.beam:567 (compile:file/2)
         dialyzer_utils.beam:109 (compile:noenv_file/2)
         dialyzer_utils.beam:184 (compile:noenv_forms/2)
         diameter_tcp.beam:[188,659,661,800,802,830,839] (ssl:_/_)
         eldap.beam:[513,585,625,962,1003,1008,1122,1124] (ssl:_/_)
         ftp.beam:[1737,2309,2327,2359,2378,2388,2512,2515,2520] (ssl:_/_)
         http_transport.beam:[109,179,214,235,256,289,337,356,379,412,490] (ssl:_/_)
         httpc_handler.beam:1837 (ssl:_/_)
         httpd_script_env.beam:66 (ssl:_/_)
         hdlt_client.beam:212 (ssl:_/_)
         hdlt_ctrl.beam:[201,305,424,995,1011] (ssh:_/_)
         hdlt_ctrl.beam:[298,333,339,345,356,369,418,447,453,459,474,988,1006,1015,1017,1023,1027,1029,1441,1461,1477,1488,1497] (ssh_sftp:_/_)
         hdlt_server.beam:150 (ssl:_/_)
         hdlt_slave.beam:[154,222,227] (ssh:_/_)
         hdlt_slave.beam:[166,177] (ssh_connection:_/_)
         orber_socket.beam:[65,116,153,254,291,307,324,332,340,348,354,367,375,383,403,468] (ssl:_/_)
         os_mon_mib.beam:[111,113,137,140] (snmp_shadow_table:_/_)
         os_mon_mib.beam:[90,99] (snmpa:_/_)
         otp_mib.beam:[109,112,116,118] (snmp_shadow_table:_/_)
         otp_mib.beam:[73,82] (snmpa:_/_)
         OTP-PUB-KEY.beam:[1633,8018,8078,8222,8272,8728,8776,8805,8853,9027,9075,9104,9152,9181,9229,9390,9450,9525,9575,9680,9757,9840,9886,10010,10047,14639,14700,14757,14799,14856,14909,14934,14976,15062,15110,15199,15247,15380,15406,15451,15488,15619,15661] ('OTP-PUB-KEY':_/_)
         PKCS-FRAME.beam:[202,414,443,514,555,606,635,706,747,895,924,1046,1087,1146,1205,1297,1368,1414,1539,1716,1807] ('PKCS-FRAME':_/_)
         pubkey_cert.beam:467 ('OTP-PUB-KEY':_/_)
         pubkey_cert.beam:1049 (pubkey_cert:_/_)
         pubkey_cert.beam:49 (pubkey_cert_records:_/_)
         pubkey_cert.beam:[453,460,462] (public_key:_/_)
         pubkey_cert_records.beam:[40,226,239,284,298] ('OTP-PUB-KEY':_/_)
         pubkey_crl.beam:578 ('OTP-PUB-KEY':_/_)
         pubkey_crl.beam:[43,70,85,311,327,413,423,475,495,636,647,673] (pubkey_cert:_/_)
         pubkey_crl.beam:[280,309,325,336,375,380,383,391,697,702] (pubkey_cert_records:_/_)
         pubkey_crl.beam:[213,227,238,477,563,565,573,660,662,664] (public_key:_/_)
         pubkey_pbe.beam:[162,184,187,190,193,196,200,206] ('PKCS-FRAME':_/_)
         pubkey_pbe.beam:[61,66,70,76] (crypto:block_decrypt/4)
         pubkey_pbe.beam:[44,49,53] (crypto:block_encrypt/4)
         pubkey_pem.beam:[79,89,146,150] (pubkey_pbe:_/_)
         pubkey_pem.beam:[145,151] (public_key:_/_)
         pubkey_ssh.beam:[198,413,447,478,606] (public_key:_/_)
         public_key.beam:[235,259,1097,1139] ('OTP-PUB-KEY':_/_)
         public_key.beam:[226,250] ('PKCS-FRAME':_/_)
         public_key.beam:[415,418] (crypto:compute_key/4)
         public_key.beam:[401,1078] (crypto:generate_key/2)
         public_key.beam:319 (crypto:private_decrypt/4)
         public_key.beam:385 (crypto:private_encrypt/4)
         public_key.beam:867 (crypto:public_decrypt/4)
         public_key.beam:863 (crypto:public_encrypt/4)
         public_key.beam:[462,465,470] (crypto:sign/4)
         public_key.beam:[837,842,850] (crypto:verify/5)
         public_key.beam:[502,503,521,546,588,593,598,635,639,648,659,676,704,739,924,931,933,935,938,942,944,951,1046,1048] (pubkey_cert:_/_)
         public_key.beam:[131,280,299,498,523,556,640,691,1082,1087,1105] (pubkey_cert_records:_/_)
         public_key.beam:[557,616,754,762,969,983,1019,1026,1035] (pubkey_crl:_/_)
         public_key.beam:[112,120,854,858] (pubkey_pem:_/_)
         public_key.beam:[389,392,783,800] (pubkey_ssh:_/_)
         public_key.beam:497 (public_key:_/_)
         snmp.beam:[235,238,241,244] (snmp_app:_/_)
         snmp.beam:247 (snmp_config:_/_)
         snmp.beam:[933,962,965] (snmp_log:_/_)
         snmp.beam:901 (snmp_misc:_/_)
         snmp.beam:[879,882] (snmp_pdus:_/_)
         snmp.beam:[890,893] (snmp_usm:_/_)
         snmp.beam:[863,868,1000,1002,1003,1004,1005,1006,1007,1008,1009,1011,1012,1013,1014,1015,1017,1018,1019,1020,1021,1022,1024,1026,1028,1031,1034,1036,1038,1040,1042,1043,1044,1047,1049,1051,1053] (snmpa:_/_)
         snmp.beam:[988,989,992,995,998] (snmpc:_/_)
         snmp.beam:[865,870] (snmpm:_/_)
         snmp_app.beam:[39,117,141,153] (snmp_app_sup:_/_)
         snmp_app.beam:62 (snmpa_app:_/_)
         snmp_app_sup.beam:103 (snmp_misc:_/_)
         snmp_community_mib.beam:[138,147,148,149,150,151,473,480,487,494,501,577,584] (snmp_conf:_/_)
         snmp_community_mib.beam:[163,456] (snmp_framework_mib:_/_)
         snmp_community_mib.beam:[316,419,432,444,452,528,541,608,628] (snmp_generic:_/_)
         snmp_community_mib.beam:[265,301,355,359] (snmp_target_mib:_/_)
         snmp_community_mib.beam:[68,73,76,98,102,108,110,115,118,120,125,171,178,250,257,261,267,272,297,303] (snmp_verbosity:_/_)
         snmp_community_mib.beam:[445,594] (snmpa_agent:_/_)
         snmp_community_mib.beam:657 (snmpa_error:_/_)
         snmp_community_mib.beam:[71,116,117,174] (snmpa_local_db:_/_)
         snmp_community_mib.beam:[182,185,232,416] (snmpa_mib_lib:_/_)
         snmp_conf.beam:[142,150,191,199,206,218,228,232,235,240,245] (snmp_verbosity:_/_)
         snmp_config.beam:1888 (snmp_conf:_/_)
         snmp_config.beam:[1073,1117] (snmp_misc:_/_)
         snmp_config.beam:1637 (snmp_target_mib:_/_)
         snmp_config.beam:2772 (snmp_usm:_/_)
         snmp_config.beam:[1703,1706,1731,1734,1760,1763,1795,1798,1891,1894,1928,1931,1955,1958,2022,2025,2099,2102] (snmpa_conf:_/_)
         snmp_config.beam:[2168,2171,2190,2193,2212,2215,2233,2236] (snmpm_conf:_/_)
         snmp_framework_mib.beam:[125,135,144,181,196,198,203,219,221,239,241,243,250] (snmp_conf:_/_)
         snmp_framework_mib.beam:[271,382,390,394,414,418,422,425,428,431,439,445,451,460,462,466,491] (snmp_generic:_/_)
         snmp_framework_mib.beam:[472,484,494] (snmp_misc:_/_)
         snmp_framework_mib.beam:194 (snmp_target_mib:_/_)
         snmp_framework_mib.beam:[94,98,106,108,110,114,119,130,179,261,269,274,282] (snmp_verbosity:_/_)
         snmp_framework_mib.beam:[86,405] (snmpa_agent:_/_)
         snmp_framework_mib.beam:512 (snmpa_error:_/_)
         snmp_framework_mib.beam:[257,262,275,276,283,399] (snmpa_local_db:_/_)
         snmp_framework_mib.beam:[289,292,437,443,449,455] (snmpa_mib_lib:_/_)
         snmp_generic.beam:92 (snmp_generic:_/_)
         snmp_generic.beam:[61,65,70,105,117,123,128,196,202,506,600,816,823] (snmp_generic_mnesia:_/_)
         snmp_generic.beam:[100,107,213,216,234,237,242,246,249,253,419,441,732,734] (snmp_verbosity:_/_)
         snmp_generic.beam:916 (snmpa_error:_/_)
         snmp_generic.beam:[63,67,72,80,82,112,120,125,130,133,199,205,416,438,512,796,818,825] (snmpa_local_db:_/_)
         snmp_generic.beam:[740,747,755,762] (snmpa_symbolic_store:_/_)
         snmp_generic_mnesia.beam:[91,92,104,105,109,123,145,205,216,217,226,229,231,244,270,317,318,319,321,323,352,374,383] (snmp_generic:_/_)
         snmp_generic_mnesia.beam:402 (snmpa_error:_/_)
         snmp_index.beam:[55,60,65,71,78,87,100,111] (snmp_verbosity:_/_)
         snmp_log.beam:884 (snmp_conf:_/_)
         snmp_log.beam:[649,659,668,678] (snmp_mini_mib:_/_)
         snmp_log.beam:[956,974,976] (snmp_misc:_/_)
         snmp_log.beam:[739,751,759,771,959] (snmp_pdus:_/_)
         snmp_log.beam:[123,157,175,225,237,249,256,264,324,329,334,342,362,370,433,465,507,546,548,571,574,581,584,589,593,598,602,1013,1042,1051] (snmp_verbosity:_/_)
         snmp_mini_mib.beam:[60,62] (snmp_misc:_/_)
         snmp_misc.beam:461 (snmp_mini_mib:_/_)
         snmp_misc.beam:[350,465] (snmp_misc:_/_)
         snmp_misc.beam:470 (snmp_pdus:_/_)
         snmp_note_store.beam:[148,357,377,444,447,450] (snmp_misc:_/_)
         snmp_note_store.beam:[317,323,327,334,340,345] (snmp_note_store:_/_)
         snmp_note_store.beam:[120,127,143,146,149,151,153,158,166,168,173,175,179,184,193,194,211,223,228,248,354,356,358,361,364] (snmp_verbosity:_/_)
         snmp_notification_mib.beam:[122,126,127,128,390,397] (snmp_conf:_/_)
         snmp_notification_mib.beam:[351,367,375,380,445,449,453] (snmp_generic:_/_)
         snmp_notification_mib.beam:286 (snmp_misc:_/_)
         snmp_notification_mib.beam:229 (snmp_target_mib:_/_)
         snmp_notification_mib.beam:[59,64,67,89,93,99,102,108,137,140,243,252,259,266,272,279,288,292] (snmp_verbosity:_/_)
         snmp_notification_mib.beam:434 (snmpa_agent:_/_)
         snmp_notification_mib.beam:476 (snmpa_error:_/_)
         snmp_notification_mib.beam:[62,138,139,145] (snmpa_local_db:_/_)
         snmp_notification_mib.beam:[150,153,187,348] (snmpa_mib_lib:_/_)
         snmp_notification_mib.beam:[207,211,221] (snmpa_target_cache:_/_)
         snmp_shadow_table.beam:119 (snmp_generic:_/_)
         snmp_shadow_table.beam:80 (snmp_misc:_/_)
         snmp_standard_mib.beam:[168,192,200,201,202,203,204,205,209] (snmp_conf:_/_)
         snmp_standard_mib.beam:[105,139,224,226,230,253,262,271,280,289,498,504,507,523,529,532,549,554,560,563,568,576] (snmp_generic:_/_)
         snmp_standard_mib.beam:[86,90,126,130,155] (snmp_verbosity:_/_)
         snmp_standard_mib.beam:[473,480] (snmpa:_/_)
         snmp_standard_mib.beam:[96,595] (snmpa_agent:_/_)
         snmp_standard_mib.beam:612 (snmpa_error:_/_)
         snmp_standard_mib.beam:[103,138,579,580,583,584,588,589,592] (snmpa_local_db:_/_)
         snmp_standard_mib.beam:[249,258,267,276,285,458,477,495,520] (snmpa_mib_lib:_/_)
         snmp_standard_mib.beam:220 (snmpa_mpd:_/_)
         snmp_target_mib.beam:[149,284,285,287,288,289,290,293,294,295,296,307,310,320,336,337,340,341,342,707,714,822,829,841,850,851,859,866,873,880,988,1015] (snmp_conf:_/_)
         snmp_target_mib.beam:[663,675,680,686,689,694,765,786,802,808,959,973,979,1049,1053,1057] (snmp_generic:_/_)
         snmp_target_mib.beam:[511,529,534] (snmp_misc:_/_)
         snmp_target_mib.beam:[131,776,972] (snmp_notification_mib:_/_)
         snmp_target_mib.beam:[81,87,91,114,118,124,126,128,130,136,270,297,351,356,462,489,496,499,509,513,517,526,532,536,540,546,551,579] (snmp_verbosity:_/_)
         snmp_target_mib.beam:1034 (snmpa_agent:_/_)
         snmp_target_mib.beam:1077 (snmpa_error:_/_)
         snmp_target_mib.beam:[85,353,354,358,359,364,370,609,652] (snmpa_local_db:_/_)
         snmp_target_mib.beam:[375,378,450,454,672,762,956] (snmpa_mib_lib:_/_)
         snmp_user_based_sm_mib.beam:[146,159,160,161,163,165,173,174,175,179,180,181,182,183,184,604,611,618,629,648,655,673,680,687] (snmp_conf:_/_)
         snmp_user_based_sm_mib.beam:[400,408,424,442,447,453,456,461,537,556,580,587,754,792,802,923,942,1068,1073,1098,1117,1139,1143,1147] (snmp_generic:_/_)
         snmp_user_based_sm_mib.beam:[1173,1189,1206] (snmp_misc:_/_)
         snmp_user_based_sm_mib.beam:[85,91,95,118,122,128,130,132,134,140,246,390,394,411,415,545,550,558,565,570,575,591,595,599] (snmp_verbosity:_/_)
         snmp_user_based_sm_mib.beam:1128 (snmpa_agent:_/_)
         snmp_user_based_sm_mib.beam:1254 (snmpa_error:_/_)
         snmp_user_based_sm_mib.beam:[89,247,248,253] (snmpa_local_db:_/_)
         snmp_user_based_sm_mib.beam:[263,266,309,320,326,332,338,344,350,439,534] (snmpa_mib_lib:_/_)
         snmp_usm.beam:[234,261] (crypto:block_decrypt/4)
         snmp_usm.beam:[219,251] (crypto:block_encrypt/4)
         snmp_usm.beam:[216,232] (snmp_misc:_/_)
         snmp_usm.beam:[157,190,236,263,279] (snmp_pdus:_/_)
         snmp_usm.beam:[181,206,225,239] (snmp_verbosity:_/_)
         snmp_view_based_acm_mib.beam:[140,147,148,149,155,156,157,158,161,162,163,164,172,173,175,177,475,484,674,713,720,727,950,957] (snmp_conf:_/_)
         snmp_view_based_acm_mib.beam:[360,362] (snmp_framework_mib:_/_)
         snmp_view_based_acm_mib.beam:[349,409,424,437,443,850,855,861,864,869,917,926,934,940,1053,1089,1094] (snmp_generic:_/_)
         snmp_view_based_acm_mib.beam:[78,84,88,111,115,121,123,128,186,191,195,200,416,421,430,435,447,451,455,470,479,488] (snmp_verbosity:_/_)
         snmp_view_based_acm_mib.beam:[255,269,286,295,308,322,359,423,661,925,1072] (snmpa_agent:_/_)
         snmp_view_based_acm_mib.beam:1139 (snmpa_error:_/_)
         snmp_view_based_acm_mib.beam:[82,187,188,196,197,209,230] (snmpa_local_db:_/_)
         snmp_view_based_acm_mib.beam:[236,239,333,337,407,547,847,915] (snmpa_mib_lib:_/_)
         snmp_view_based_acm_mib.beam:[192,222,285,296,342,552,576,741,751,754,757,766,769,778,790,799,1060] (snmpa_vacm:_/_)
         snmpa.beam:[895,902,918,923,940,944,962,965,984,987,1008,1012,1025,1036,1041,1047,1051,1056,1059,1063,1066,1070,1073,1076,1086] (snmp:_/_)
         snmpa.beam:1081 (snmp_log:_/_)
         snmpa.beam:830 (snmp_misc:_/_)
         snmpa.beam:[863,866,869] (snmp_standard_mib:_/_)
         snmpa.beam:[853,856] (snmp_usm:_/_)
         snmpa.beam:[176,177,178,179,180,184,186,188,190,196,198,271,272,274,275,279,298,303,304,324,336,356,368,372,378,564,567,570,573,580,587,594,601,608,613,618,625,632,639,646,666,672,678,684,689,695,808,818,821,839,846,1093,1106] (snmpa_agent:_/_)
         snmpa.beam:167 (snmpa_app:_/_)
         snmpa.beam:806 (snmpa_discovery_handler:_/_)
         snmpa.beam:[182,194] (snmpa_local_db:_/_)
         snmpa.beam:542 (snmpa_mib_lib:_/_)
         snmpa.beam:[181,192,208,212,215,218,221,228,231,234,237,240,243,246,249] (snmpa_symbolic_store:_/_)
         snmpa_acm.beam:138 (snmp_community_mib:_/_)
         snmpa_acm.beam:121 (snmp_conf:_/_)
         snmpa_acm.beam:121 (snmp_target_mib:_/_)
         snmpa_acm.beam:[126,134,144,170,179,247,323,331] (snmp_verbosity:_/_)
         snmpa_acm.beam:[283,348] (snmp_view_based_acm_mib:_/_)
         snmpa_acm.beam:248 (snmpa:_/_)
         snmpa_acm.beam:243 (snmpa_agent:_/_)
         snmpa_acm.beam:157 (snmpa_mpd:_/_)
         snmpa_acm.beam:[147,191] (snmpa_vacm:_/_)
         snmpa_agent.beam:[891,914,4246] (snmp_framework_mib:_/_)
         snmpa_agent.beam:[583,2913,2984,3999,4000,4009,4031,4071,4079,4163,4488,4491] (snmp_misc:_/_)
         snmpa_agent.beam:[4226,4356] (snmp_note_store:_/_)
         snmpa_agent.beam:3700 (snmp_pdus:_/_)
         snmpa_agent.beam:2315 (snmp_target_mib:_/_)
         snmpa_agent.beam:2483 (snmp_user_based_sm_mib:_/_)
         snmpa_agent.beam:[347,358,400,403,407,410,413,416,422,430,437,440,443,446,451,456,461,467,470,473,476,801,807,819,823,831,839,853,861,875,884,897,906,920,928,946,950,954,958,961,970,985,997,1009,1012,1020,1023,1030,1038,1041,1048,1062,1065,1072,1086,1089,1099,1112,1123,1130,1138,1146,1153,1167,1174,1190,1211,1217,1234,1239,1244,1249,1253,1257,1261,1268,1292,1306,1310,1323,1338,1349,1361,1366,1372,1377,1391,1392,1404,1580,1625,1629,1634,1637,1641,1645,1649,1656,1659,1758,1765,1773,1799,1828,1832,1843,1850,1860,1869,1883,1894,1900,1909,1916,1934,1942,1949,1953,1969,1991,2023,2056,2065,2071,2076,2089,2093,2097,2102,2109,2121,2125,2129,2134,2141,2158,2166,2173,2182,2187,2247,2255,2289,2297,2308,2346,2359,2367,2378,2385,2399,2418,2423,2433,2443,2453,2463,2468,2473,2481,2487,2532,2537,2539,2546,2552,2558,2563,2570,2575,2588,2598,2605,2711,2735,2813,2817,2856,2875,2890,2906,2925,2953,2960,2975,3188,3234,3254,3304,3345,3356,3626,3650,3661,3665,3669,3678,3702,3743,3769,3791,3830,3952,4091,4188,4192,4201] (snmp_verbosity:_/_)
         snmpa_agent.beam:[1175,1978,2739,3444] (snmpa_acm:_/_)
         snmpa_agent.beam:1885 (snmpa_agent:_/_)
         snmpa_agent.beam:[4416,4419] (snmpa_error:_/_)
         snmpa_agent.beam:[1640,4372] (snmpa_local_db:_/_)
         snmpa_agent.beam:[982,986,1202,1212,1219,1221,1235,1240,1245,1250,1254,1258,1314,1318,1448,1450,1452,1454,1456,1458,1460,1462,1464,1466,1469,1644,2494,2509,2677,3384,3386,3388,3496,4221,4231,4380] (snmpa_mib:_/_)
         snmpa_agent.beam:[405,434,465,1434,1435] (snmpa_misc_sup:_/_)
         snmpa_agent.beam:4388 (snmpa_mpd:_/_)
         snmpa_agent.beam:[2930,3400,3541,3841,3844] (snmpa_svbl:_/_)
         snmpa_agent.beam:[1648,4364] (snmpa_symbolic_store:_/_)
         snmpa_agent.beam:[1759,1811,1852,1862,2063,2151,2160,2253,2274,2352] (snmpa_trap:_/_)
         snmpa_agent.beam:1636 (snmpa_vacm:_/_)
         snmpa_agent_sup.beam:78 (snmpa_agent:_/_)
         snmpa_app.beam:121 (snmp_app_sup:_/_)
         snmpa_conf.beam:282 (snmp_community_mib:_/_)
         snmpa_conf.beam:[887,890,893] (snmp_config:_/_)
         snmpa_conf.beam:207 (snmp_framework_mib:_/_)
         snmpa_conf.beam:652 (snmp_notification_mib:_/_)
         snmpa_conf.beam:350 (snmp_standard_mib:_/_)
         snmpa_conf.beam:[476,505,587] (snmp_target_mib:_/_)
         snmpa_conf.beam:741 (snmp_user_based_sm_mib:_/_)
         snmpa_conf.beam:844 (snmp_view_based_acm_mib:_/_)
         snmpa_discovery_handler.beam:30 (snmp_misc:_/_)
         snmpa_error_logger.beam:49 (snmp_misc:_/_)
         snmpa_local_db.beam:[1014,1019,1020,1021,1024,1049,1063,1065,1076,1088,1089,1101,1102,1113,1114,1122,1130] (snmp_generic:_/_)
         snmpa_local_db.beam:1145 (snmp_misc:_/_)
         snmpa_local_db.beam:[137,152,155,166,169,203,205,345,349,354,376,381,387,392,397,406,411,419,424,433,438,445,450,458,463,467,478,483,486,491,501,517,526,530,535,540,546,551,557,565,572,577,587,597,598,607,617,628,666,671,674,677,683,691,695,702,707,715,717,719,721,724,878,885,899,1015] (snmp_verbosity:_/_)
         snmpa_local_db.beam:[1200,1203] (snmpa_error:_/_)
         snmpa_mib.beam:[920,926,929] (snmp_misc:_/_)
         snmpa_mib.beam:[279,280,312,317,322,324,335,347,360,364,368,371,386,391,399,406,412,417,423,434,438,443,445,450,461,465,479,486,505,513,522,529,537,542,552,562,569,582,594,605,620,629,633,642,643,652,662,672,678,847] (snmp_verbosity:_/_)
         snmpa_mib.beam:956 (snmpa_error:_/_)
         snmpa_mib_data_tttn.beam:[567,567] (snmp:_/_)
         snmpa_mib_data_tttn.beam:[156,157,277,511,524,1171] (snmp_misc:_/_)
         snmpa_mib_data_tttn.beam:[161,171,181,191,201,203,216,223,232,235,245,259,279,329,334,345,352,362,366,373,378,384,397,503,541,576,613,617,621,625,629,633,636,653,657,661,670,675,679,682,689,694,706,714,720,731,739,747,752,755,758,766,772,802,808,814,822,828,832,840,856,863,868,883,891,901,912,950,964,984,1015,1020,1023,1026,1030,1039,1050,1302,1312,1323,1370,1375,1405,1412] (snmp_verbosity:_/_)
         snmpa_mib_data_tttn.beam:[820,874,896,943,959,975] (snmpa_acm:_/_)
         snmpa_mib_data_tttn.beam:[346,353,374,1339,1340,1341,1342,1344,1381,1382,1383,1384,1385] (snmpa_symbolic_store:_/_)
         snmpa_mib_lib.beam:218 (snmp_generic:_/_)
         snmpa_mib_lib.beam:[163,169,176,195,241] (snmp_generic_mnesia:_/_)
         snmpa_mib_lib.beam:[43,46,51,54,68,72,75,80,224,231] (snmp_verbosity:_/_)
         snmpa_mib_lib.beam:247 (snmpa_error:_/_)
         snmpa_mib_lib.beam:[47,55,165,171,178,207,244] (snmpa_local_db:_/_)
         snmpa_mib_storage_dets.beam:[67,68,69,70] (snmp_misc:_/_)
         snmpa_mib_storage_dets.beam:[108,119,134,145,162,173,185,198,210,217,220,231,242,261,268,274,281,284] (snmp_verbosity:_/_)
         snmpa_mib_storage_ets.beam:[75,76] (snmp_misc:_/_)
         snmpa_mib_storage_ets.beam:[72,77,81,85,92,98,114,124,130,146,160,163,175,191,201,204,215,227,239,253,265,277,290,304] (snmp_verbosity:_/_)
         snmpa_mib_storage_ets.beam:342 (snmpa_error:_/_)
         snmpa_mib_storage_mnesia.beam:[276,279] (snmp_misc:_/_)
         snmpa_mib_storage_mnesia.beam:[65,71,74,83,90,93,116,127,141,158,169,187,205,229] (snmp_verbosity:_/_)
         snmpa_mpd.beam:223 (snmp_community_mib:_/_)
         snmpa_mpd.beam:[202,203,205,206,1118] (snmp_conf:_/_)
         snmpa_mpd.beam:[128,138,610,884,1205,1392] (snmp_framework_mib:_/_)
         snmpa_mpd.beam:[310,494,672,764,767,768,769,1208] (snmp_misc:_/_)
         snmpa_mpd.beam:[249,254,385,422,1050,1305] (snmp_note_store:_/_)
         snmpa_mpd.beam:[143,228,333,377,598,619,630,662,847,855,889,914,986] (snmp_pdus:_/_)
         snmpa_mpd.beam:1389 (snmp_target_mib:_/_)
         snmpa_mpd.beam:[77,147,155,163,172,177,182,244,265,289,299,312,320,325,335,351,380,397,409,434,446,467,471,512,561,570,574,587,592,645,685,721,728,758,784,812,901,1015,1029,1038,1053,1127,1132,1138,1144,1149,1156,1161,1166,1215,1245,1260,1270,1279,1285] (snmp_verbosity:_/_)
         snmpa_mpd.beam:[1521,1524] (snmpa_error:_/_)
         snmpa_net_if.beam:[292,1445] (snmp_conf:_/_)
         snmpa_net_if.beam:118 (snmp_framework_mib:_/_)
         snmpa_net_if.beam:[242,251,279,286,1350,1362,1372,1373] (snmp_log:_/_)
         snmpa_net_if.beam:[987,1020,1480,1490,1493,1496,1499,1502,1505,1508,1511,1514,1517,1520,1523,1526] (snmp_misc:_/_)
         snmpa_net_if.beam:[169,173,177,180,184,187,200,213,217,227,245,253,299,306,310,318,334,337,360,391,401,411,423,436,447,457,482,488,494,499,505,510,511,519,562,568,618,625,694,701,707,715,790,794,808,813,819,874,885,900,949,963,972,984,1002,1019,1024,1057,1073,1184,1188,1221,1343] (snmp_verbosity:_/_)
         snmpa_net_if.beam:[1540,1543] (snmpa_error:_/_)
         snmpa_net_if.beam:[203,459,683,871,904,967,1008] (snmpa_mpd:_/_)
         snmpa_net_if.beam:267 (snmpa_network_interface_filter:_/_)
         snmpa_network_interface_filter.beam:56 (snmp_misc:_/_)
         snmpa_notification_delivery_info_receiver.beam:36 (snmp_misc:_/_)
         snmpa_set.beam:[62,85,89,105,127,160] (snmp_verbosity:_/_)
         snmpa_set.beam:64 (snmpa_acm:_/_)
         snmpa_set.beam:[141,189,221] (snmpa_agent:_/_)
         snmpa_set.beam:245 (snmpa_error:_/_)
         snmpa_set.beam:[129,131,164,217] (snmpa_set_lib:_/_)
         snmpa_set.beam:[238,241] (snmpa_svbl:_/_)
         snmpa_set_lib.beam:179 (snmp_misc:_/_)
         snmpa_set_lib.beam:[81,86,146,161,174,184,191,197,204,231,408,410] (snmp_verbosity:_/_)
         snmpa_set_lib.beam:[354,356] (snmpa_agent:_/_)
         snmpa_set_lib.beam:[335,337,339] (snmpa_svbl:_/_)
         snmpa_supervisor.beam:[547,549] (snmp_framework_mib:_/_)
         snmpa_supervisor.beam:[332,654,657] (snmp_misc:_/_)
         snmpa_supervisor.beam:[183,185,188,193,200,205,210,215,219,224,322,331,336,341,346,364,379,392,397,404,409,416,422,425,435,441,447,474,479,490,494,522,532,542,544,546,548,550,552,554,558,562,567,571,574] (snmp_verbosity:_/_)
         snmpa_supervisor.beam:[95,121] (snmpa:_/_)
         snmpa_supervisor.beam:[167,170] (snmpa_agent_sup:_/_)
         snmpa_supervisor.beam:365 (snmpa_vacm:_/_)
         snmpa_svbl.beam:83 (snmp_pdus:_/_)
         snmpa_svbl.beam:52 (snmp_verbosity:_/_)
         snmpa_svbl.beam:53 (snmpa_mib:_/_)
         snmpa_symbolic_store.beam:[344,345,498,726,729] (snmp_misc:_/_)
         snmpa_symbolic_store.beam:[342,352,360,364,369,371,376,378,383,385,390,392,397,399,404,406,411,413,418,420,425,427,432,439,445,451,453,457,462,477,486,496,500,512,519,536,545,547,559,566,568,580,588,589,598,608,619,695,700] (snmp_verbosity:_/_)
         snmpa_symbolic_store.beam:750 (snmpa_error:_/_)
         snmpa_target_cache.beam:845 (snmp_misc:_/_)
         snmpa_target_cache.beam:[166,182,219,236,257,274,292,308,309,316,334,342,351,370,392,409,414,422,435,444,450,458,467,477,483,496,499,516,527,555,561,575,598] (snmp_verbosity:_/_)
         snmpa_target_cache.beam:865 (snmpa_error:_/_)
         snmpa_trap.beam:[707,728] (snmp_community_mib:_/_)
         snmpa_trap.beam:[347,352,639,826,944,993,1099] (snmp_framework_mib:_/_)
         snmpa_trap.beam:[484,487] (snmp_notification_mib:_/_)
         snmpa_trap.beam:[593,782] (snmp_standard_mib:_/_)
         snmpa_trap.beam:[502,521] (snmp_target_mib:_/_)
         snmpa_trap.beam:[134,142,150,373,399,404,483,486,489,509,527,555,576,665,669,692,705,712,719,726,733,741,748,756,762,770,783,797,809,832,850,858,870,880,885,920,931,946,960,982,1011,1020,1024,1030,1045,1051,1063,1073,1088,1093] (snmp_verbosity:_/_)
         snmpa_trap.beam:[402,1184,1195,1201,1213] (snmpa_acm:_/_)
         snmpa_trap.beam:[402,423,1213] (snmpa_agent:_/_)
         snmpa_trap.beam:1254 (snmpa_error:_/_)
         snmpa_trap.beam:263 (snmpa_mib:_/_)
         snmpa_trap.beam:[306,313] (snmpa_mpd:_/_)
         snmpa_trap.beam:[136,159,164] (snmpa_symbolic_store:_/_)
         snmpa_trap.beam:[644,675,987,1017,1035] (snmpa_trap:_/_)
         snmpa_trap.beam:1249 (snmpa_vacm:_/_)
         snmpa_usm.beam:[70,438,656,660,661,718,721] (snmp_framework_mib:_/_)
         snmpa_usm.beam:[166,197,391,492,579,611,738,753] (snmp_misc:_/_)
         snmpa_usm.beam:[79,399,588,618] (snmp_pdus:_/_)
         snmpa_usm.beam:[101,113,455,545] (snmp_user_based_sm_mib:_/_)
         snmpa_usm.beam:[625,628,631,634,638,662,670] (snmp_usm:_/_)
         snmpa_usm.beam:[77,90,96,100,111,118,132,135,142,149,163,168,185,192,224,230,232,246,263,273,287,297,312,329,346,350,353,359,365,373,379,403,417,429,445,465,479,488,505,514,525,555,581,584,587,590,594,617] (snmp_verbosity:_/_)
         snmpa_usm.beam:[698,701,704] (snmpa_agent:_/_)
         snmpa_vacm.beam:[66,74,84,93,100,108,122,215,227,236] (snmp_verbosity:_/_)
         snmpa_vacm.beam:[75,87,125,146,159] (snmp_view_based_acm_mib:_/_)
         snmpa_vacm.beam:448 (snmpa_error:_/_)
         snmpa_vacm.beam:80 (snmpa_mpd:_/_)
         snmpc.beam:169 (snmpc:_/_)
         snmpc.beam:[41,49,300,330,333,356,393,399,402,442,445,452,490,498,528,558,565,576,578,609,638,639,642,649,660,662,689,717,721,738,740,752,762,766,767,784,796,797,798,807,809,810,823,834,845,843,853,860,863,861,873,881,892,894,904,915,922,931,943,953,962,963,972,984,985,1006,1018,1030,1031,1052,1065,1077,1081,1088,1089,1097,1109,1121,1126,1133,1134,1142,1151,1166,1170,1171,1176,1178,1188,1193,1200,1200,1204,1206,1213,1241,1254,1255,1258,1269,1274,1276,1300,1344,1355,1362,1367,1375,1381,1391,1397,1400,1428,1441,1447,1452,1469,1476,1480,1485,1501,1513,1530,1533] (snmpc_lib:_/_)
         snmpc.beam:1519 (snmpc_mib_gram:_/_)
         snmpc.beam:[52,55] (snmpc_mib_to_hrl:_/_)
         snmpc.beam:[129,243,1463,1464,1466,1467] (snmpc_misc:_/_)
         snmpc.beam:[1497,1504,1516] (snmpc_tok:_/_)
         snmpc_lib.beam:[412,413,659,828,864,880,920,922,943,951,956,961,965,968,971,977,992,996,1019,1033,1037,1046,1121,1125,1129,1141,1146,1191,1216,1219,1227,1239,1387,1400,1481,1483,1624,1634,1638,1648,1654,1658,1663,1701,1712,1722,1730,1740,1746,1751] (snmpc_lib:_/_)
         snmpc_lib.beam:[169,350,351,406,407,539,541,584,612,615,621,635,637,638,650,667,857,979,987,1329,1354,1478,1519,1531,1762] (snmpc_misc:_/_)
         snmpc_mib_gram.beam:[1033,1038,1044,1166,1182,1182] (snmpc_lib:_/_)
         snmpc_mib_gram.beam:1053 (snmpc_misc:_/_)
         snmpc_mib_to_hrl.beam:[48,54,59,65,77,79,85,94,100,104,113,118,124,132,145,156,230,266,333,352] (snmpc_lib:_/_)
         snmpc_mib_to_hrl.beam:[52,240,250] (snmpc_misc:_/_)
         snmpc_misc.beam:58 (snmp_misc:_/_)
         snmpm.beam:[822,828,844,849,866,870,888,891,910,913,934,937,950,961,965,971,975,980,983,987,990,994,997,1000,1005] (snmp:_/_)
         snmpm.beam:403 (snmp_conf:_/_)
         snmpm.beam:165 (snmp_config:_/_)
         snmpm.beam:[253,254,1029] (snmp_misc:_/_)
         snmpm.beam:[239,248,1245] (snmpm:_/_)
         snmpm.beam:[260,275,279,283,287,300,308,340,358,401,412,428,431,438,441,448,452,456,460,463,466,1023,1284,1287] (snmpm_config:_/_)
         snmpm.beam:[243,267,271,293,302,304,306,309,310,311,328,334,337,498,535,567,600,633,666,710,761,796,1009,1013,1017] (snmpm_server:_/_)
         snmpm.beam:[184,194,199] (snmpm_supervisor:_/_)
         snmpm_conf.beam:[349,352,355] (snmp_config:_/_)
         snmpm_conf.beam:[177,243,314] (snmpm_config:_/_)
         snmpm_config.beam:[304,1749,1794,1809,1850,1868,1893,1895,1897,1903,1906,1908,1919,1921,1927,2111,2119,2127,2140,2150,2168,2186,2196,2214,2263,2281,2286,2288,2292,2308,2310,2321,2336,2338,2344,2352,3017,3071,3321] (snmp_conf:_/_)
         snmpm_config.beam:[227,232,697,715,1086,1302,1440,2234,2491,3251,3253,3369,3373] (snmp_misc:_/_)
         snmpm_config.beam:[352,1033,1104,1128,1134,1153,1158,1161,1164,1179,1183,1197,1205,1210,1215,1226,1623,1692,1746,1797,1803,1825,1939,1944,1951,1959,1962,2025,2046,2347,2371,2385,2391,2399,2407,2417,2427,2433,2441,2450,2455,2460,2465,2475,2485,2490,2496,2501,2506,2511,2516,2531,2547,2578,2588,2604,2677,2682,2685,2688,2694,2702,2706,2713,2718,2723,2730,2733,2762,2773,2780,2786,2790,2794,2805,2835,2841,2851,2860,2874,2898,2913,2935,2943,2953,2963,2976,2985,3188,3221,3250] (snmp_verbosity:_/_)
         snmpm_mpd.beam:[167,168,873] (snmp_conf:_/_)
         snmpm_mpd.beam:[368,524,657,845,846,847,848] (snmp_misc:_/_)
         snmpm_mpd.beam:[301,327,550] (snmp_note_store:_/_)
         snmpm_mpd.beam:[103,180,256,291,478,504,597,649,734,784,813] (snmp_pdus:_/_)
         snmpm_mpd.beam:[70,79,124,133,139,145,158,178,182,189,196,216,231,237,244,248,260,266,294,296,305,310,319,348,356,365,390,456,468,473,500,522,535,537,539,545,553,596,619,688,693,812,872] (snmp_verbosity:_/_)
         snmpm_mpd.beam:[74,75,858,868,887,896,899,912,941,950,994] (snmpm_config:_/_)
         snmpm_mpd.beam:[928,945] (snmpm_usm:_/_)
         snmpm_net_if.beam:[324,468,1117] (snmp_conf:_/_)
         snmpm_net_if.beam:[416,425,453,454,470] (snmp_log:_/_)
         snmpm_net_if.beam:[815,875,1171] (snmp_misc:_/_)
         snmpm_net_if.beam:[255,260,272,277,280,292,308,313,398,401,419,427,489,494,499,504,508,513,530,541,549,570,581,586,633,705,711,759,765,771,776,781,785,792,799,865,937,940,979,1002,1006,1170] (snmp_verbosity:_/_)
         snmpm_net_if.beam:[245,252,258,263,266,275,279,402,403,404,405,408,1024,1046,1098,1114,1251,1255,1266] (snmpm_config:_/_)
         snmpm_net_if.beam:[259,697,861,934] (snmpm_mpd:_/_)
         snmpm_net_if.beam:385 (snmpm_network_interface_filter:_/_)
         snmpm_net_if_mt.beam:[324,468,666,828,910,1117] (snmp_conf:_/_)
         snmpm_net_if_mt.beam:[416,425,438,453,454,470] (snmp_log:_/_)
         snmpm_net_if_mt.beam:[815,875,1171] (snmp_misc:_/_)
         snmpm_net_if_mt.beam:[255,260,272,277,280,292,308,313,398,401,419,427,489,494,499,504,508,513,530,541,549,570,581,586,608,615,633,705,711,759,765,771,776,781,785,792,799,865,937,940,979,1002,1006,1170] (snmp_verbosity:_/_)
         snmpm_net_if_mt.beam:[245,252,258,263,266,275,279,402,403,404,405,408,1024,1046,1098,1114,1251,1255,1266] (snmpm_config:_/_)
         snmpm_net_if_mt.beam:[259,697,861,934] (snmpm_mpd:_/_)
         snmpm_net_if_mt.beam:385 (snmpm_network_interface_filter:_/_)
         snmpm_network_interface_filter.beam:55 (snmp_misc:_/_)
         snmpm_server.beam:[1374,1423,1479,1527,1603,2936,3297,3355] (snmp_misc:_/_)
         snmpm_server.beam:[945,3479] (snmp_note_store:_/_)
         snmpm_server.beam:3066 (snmp_pdus:_/_)
         snmpm_server.beam:[472,488,541,567,572,580,584,591,595,599,609,613,617,622,630,633,643,648,658,661,668,670,678,694,709,726,743,759,776,793,808,821,831,844,856,870,883,896,905,911,922,932,938,944,949,954,959,964,969,975,990,996,1001,1006,1012,1018,1024,1039,1047,1056,1064,1071,1107,1129,1139,1144,1149,1164,1182,1192,1197,1202,1218,1239,1251,1256,1261,1277,1296,1306,1311,1316,1332,1350,1359,1364,1381,1399,1408,1413,1430,1453,1464,1469,1485,1503,1512,1517,1534,1542,1548,1555,1560,1581,1586,1614,1627,1661,1676,1680,1691,1711,1750,1770,1788,1832,1847,1857,1868,1882,1915,1943,1961,1972,1987,2000,2022,2033,2037,2057,2078,2149,2163,2180,2190,2227,2249,2264,2279,2297,2315,2327,2346,2357,2369,2401,2427,2442,2458,2480,2499,2513,2517,2542,2551,2562,2572,2607,2642,2662,2677,2683,2696,2704,2722,2750,2764,2779,2800,2819,2832,2897,2913,2917,2922,2970,2981,3276] (snmp_verbosity:_/_)
         snmpm_server.beam:[214,535,538,544,559,573,574,592,660,663,669,912,914,923,925,1057,1073,1097,1635,1644,1693,1717,1719,1724,1736,1801,1812,1877,1885,1892,1917,2009,2045,2066,2178,2181,2193,2196,2229,2286,2304,2316,2353,2355,2366,2372,2403,2468,2487,2500,2558,2560,2575,2578,2609,2692,2694,2707,2725,2788,2807,2820,2908,2915,2919,2924,3082,3191,3239,3376,3379,3397,3459,3467] (snmpm_config:_/_)
         snmpm_server.beam:[578,593,1109,1110] (snmpm_misc_sup:_/_)
         snmpm_server.beam:3185 (snmpm_mpd:_/_)
         snmpm_server.beam:[3304,3310,3320,3325,3334,3338,3348,3351] (snmpm_server:_/_)
         snmpm_server_sup.beam:89 (snmp_misc:_/_)
         snmpm_supervisor.beam:94 (snmp_misc:_/_)
         snmpm_usm.beam:[141,261,340,365,389,476] (snmp_misc:_/_)
         snmpm_usm.beam:[71,272,371,395] (snmp_pdus:_/_)
         snmpm_usm.beam:[403,406,409,412,415,425,433] (snmp_usm:_/_)
         snmpm_usm.beam:[69,82,87,99,113,118,135,172,174,177,179,181,201,203,205,207,217,229,247,309,332,337,349,358] (snmp_verbosity:_/_)
         snmpm_usm.beam:[88,101,314,418,437,443,447,451,460,464,468,473,477,480,512,516,527] (snmpm_config:_/_)
         snmp_ex2_manager.beam:128 (snmp_config:_/_)
         snmp_ex2_manager.beam:[136,144,185,189,193,197,201,205,220] (snmpm:_/_)
         ssh.beam:[365,379,406] (ssh_acceptor:_/_)
         ssh.beam:[375,439] (ssh_acceptor_sup:_/_)
         ssh.beam:[248,249] (ssh_channel:_/_)
         ssh.beam:[242,244] (ssh_connection:_/_)
         ssh.beam:[95,115,132,140,148] (ssh_connection_handler:_/_)
         ssh.beam:[175,194,198,208,210,212,356,374,420,438] (ssh_system_sup:_/_)
         ssh.beam:[259,812] (ssh_transport:_/_)
         ssh.beam:[361,425] (sshd_sup:_/_)
         ssh_acceptor.beam:[110,114,118] (ssh_acceptor:_/_)
         ssh_acceptor.beam:133 (ssh_connection_handler:_/_)
         ssh_acceptor.beam:131 (ssh_subsystem_sup:_/_)
         ssh_acceptor.beam:[125,130] (ssh_system_sup:_/_)
         ssh_auth.beam:[614,623] (public_key:_/_)
         ssh_auth.beam:[200,441,501] (ssh_connection_handler:_/_)
         ssh_auth.beam:560 (ssh_message:_/_)
         ssh_auth.beam:578 (ssh_no_io:_/_)
         ssh_auth.beam:[50,112,130,143,148,154,156,172,189,194,209,222,225,247,256,277,282,304,308,323,367,376,396,419,423,427,437,537,537] (ssh_transport:_/_)
         ssh_channel.beam:[245,259,282,297,370] (ssh_connection:_/_)
         ssh_cli.beam:[101,107,124,134,136,137,142,453] (ssh_connection:_/_)
         ssh_cli.beam:[485,513] (ssh_connection_handler:_/_)
         ssh_connection.beam:[261,286,288,300,301,308,318,329,336,339,369,372,387,389,399,465,479,496,508,521,535,543,573,588,605,624,639,676,684,696,849,953,999,1005] (ssh_channel:_/_)
         ssh_connection.beam:816 (ssh_channel_sup:_/_)
         ssh_connection.beam:[74,91,102,111,134,143,152,163,174,183,220,226,231,1015] (ssh_connection_handler:_/_)
         ssh_connection.beam:860 (ssh_sftpd:_/_)
         ssh_connection.beam:814 (ssh_subsystem_sup:_/_)
         ssh_connection_handler.beam:[666,684,696,713,722,781,806,815,833] (ssh_auth:_/_)
         ssh_connection_handler.beam:[393,981,987,995,1007,1049,1059,1066,1107,1125,1139,1149,1159,1162,1644,1654,1666,1664,1676,1716,1771,1868] (ssh_channel:_/_)
         ssh_connection_handler.beam:[879,999,1009,1076,1101,1109,1121,1161,1582,1610,1647,1656] (ssh_connection:_/_)
         ssh_connection_handler.beam:1442 (ssh_connection_sup:_/_)
         ssh_connection_handler.beam:1194 (ssh_message:_/_)
         ssh_connection_handler.beam:1454 (ssh_system_sup:_/_)
         ssh_connection_handler.beam:[448,529,560,569,585,586,600,602,607,613,618,623,629,631,636,644,646,654,663,871,938,959,1181,1189,1482,1484,1498,1509] (ssh_transport:_/_)
         ssh_connection_handler.beam:113 (sshc_sup:_/_)
         ssh_daemon_channel.beam:[52,55,58,61,64,67,69] (ssh_channel:_/_)
         ssh_file.beam:[139,143,145,147,282,308,316] (public_key:_/_)
         ssh_file.beam:227 (ssh_connection:_/_)
         ssh_info.beam:115 (ssh_acceptor:_/_)
         ssh_info.beam:[151,162] (ssh_channel:_/_)
         ssh_info.beam:[87,152,163,240] (ssh_connection_handler:_/_)
         ssh_message.beam:[249,275,283,448,478,491,577] (public_key:_/_)
         ssh_message.beam:[237,237,237,237,237,237,237,237,237,237,238,238,238,238,238,238,238,238,238,238,242,251,264,264,267,277,280,285] (ssh_bits:_/_)
         ssh_no_io.beam:[34,43,52,61] (ssh_connection_handler:_/_)
         ssh_sftp.beam:[112,764,773,860] (ssh:_/_)
         ssh_sftp.beam:[128,153,828,837,882,901,949] (ssh_channel:_/_)
         ssh_sftp.beam:508 (ssh_connection:_/_)
         ssh_sftp.beam:[126,151,569,593,598,606,615,629,650,672,685,703,708,713,719,724,729,734,739,744,749,755,822,891,906,961] (ssh_xfer:_/_)
         ssh_sftpd.beam:[91,98] (ssh:_/_)
         ssh_sftpd.beam:968 (ssh_connection:_/_)
         ssh_sftpd.beam:[532,578,921] (ssh_sftp:_/_)
         ssh_sftpd.beam:[234,246,263,267,279,284,299,303,321,332,342,346,345,376,386,389,423,435,459,483,490,503,511,564,577,649,656,657,673,676,692,691,879,909,912,920] (ssh_xfer:_/_)
         ssh_shell.beam:[84,155,161] (ssh_connection:_/_)
         ssh_system_sup.beam:[80,84] (ssh_subsystem_sup:_/_)
         ssh_system_sup.beam:[63,64,68] (sshd_sup:_/_)
         ssh_transport.beam:[1367,1373,1379,1384] (crypto:block_decrypt/4)
         ssh_transport.beam:[1221,1227,1233,1239,1728] (crypto:block_encrypt/4)
         ssh_transport.beam:1676 (crypto:compute_key/4)
         ssh_transport.beam:1671 (crypto:generate_key/2)
         ssh_transport.beam:[1234,1240,1380,1385] (crypto:next_iv/2)
         ssh_transport.beam:[1389,1393,1397] (crypto:stream_decrypt/2)
         ssh_transport.beam:[1244,1248,1252,1725] (crypto:stream_encrypt/2)
         ssh_transport.beam:[1169,1176,1183,1190,1197,1204,1313,1320,1327,1334,1341,1348,1725] (crypto:stream_init/3)
         ssh_transport.beam:[427,462,733,1086,1087,1090,1091,1094,1098,1099,1104,1105,1107,1591,1598,1605] (public_key:_/_)
         ssh_transport.beam:[224,969,977,1092,1092,1576,1594,1594,1594,1601,1601,1601,1612,1612,1612,1617,1617,1617,1617,1617] (ssh_bits:_/_)
         ssh_transport.beam:[279,292,375,400,408,438,473,481,499,540,547,575,583,590,626,651,659,674,929] (ssh_connection_handler:_/_)
         ssh_transport.beam:[944,949] (ssh_message:_/_)
         ssh_xfer.beam:[61,70] (ssh:_/_)
         ssh_xfer.beam:[81,278,288,300,312,331,338,345] (ssh_connection:_/_)
         sshd_sup.beam:61 (ssh_acceptor_sup:_/_)
         sshd_sup.beam:[48,60] (ssh_system_sup:_/_)
         dtls.beam:[51,60,70,84,103,113] (ssl:_/_)
         dtls_connection.beam:[72,86] (dtls_connection_sup:_/_)
         dtls_connection.beam:[150,201,247,272,308,393,459] (dtls_handshake:_/_)
         dtls_connection.beam:[205,466,540,578,594] (dtls_record:_/_)
         dtls_connection.beam:[134,535] (ssl_alert:_/_)
         dtls_connection.beam:[74,75,88,89,176,200,220,250,258,274,276,283,288,293,298,330,334,337,353,373,376,379,383,400,405,420,433,443,623,633,725,727] (ssl_connection:_/_)
         dtls_connection.beam:[147,206,462,703,710,715] (ssl_handshake:_/_)
         dtls_connection.beam:[103,508] (ssl_record:_/_)
         dtls_connection.beam:604 (ssl_socket:_/_)
         dtls_handshake.beam:[62,87,178] (dtls_record:_/_)
         dtls_handshake.beam:[67,182,209,223] (dtls_v1:_/_)
         dtls_handshake.beam:192 (ssl_cipher:_/_)
         dtls_handshake.beam:[65,67,74,98,180,183,183,185,193,208,221,455,463,472,479,491] (ssl_handshake:_/_)
         dtls_handshake.beam:[63,75] (ssl_record:_/_)
         dtls_handshake.beam:70 (ssl_session:_/_)
         dtls_record.beam:[156,172,189,212,485] (dtls_v1:_/_)
         dtls_record.beam:[70,72,153,156,169,172,189,192,212,215,217,439] (ssl_record:_/_)
         dtls_v1.beam:[29,32,36] (tls_v1:_/_)
         inet6_tls_dist.beam:[28,31,34,37,40,43,46] (inet_tls_dist:_/_)
         inet_tls_dist.beam:[62,68,96,284] (ssl_tls_dist_proxy:_/_)
         ssl.beam:613 (dtls_v1:_/_)
         ssl.beam:[414,417,420,622,622,626,626,1143,1143,1145,1149,1154,1154,1160,1164] (ssl_cipher:_/_)
         ssl.beam:[113,187,211,223,238,251,264,267,277,291,303,317,371,387,429,453,507,527,551,564,635] (ssl_connection:_/_)
         ssl.beam:574 (ssl_manager:_/_)
         ssl.beam:[104,105,110,110,111,152,152,153,180,181,221,231,232,236,236,237,361,363,432,460,493,496,515,518,1107,1107] (ssl_socket:_/_)
         ssl.beam:[538,539,621,625,943,956,967,981] (tls_record:_/_)
         ssl.beam:[730,1034,1041] (tls_v1:_/_)
         ssl_alert.beam:48 (ssl_record:_/_)
         ssl_app.beam:32 (ssl_sup:_/_)
         ssl_certificate.beam:[60,62,64,96,99,192,194,196,225,238,242,282,297,310] (public_key:_/_)
         ssl_certificate.beam:[77,107,116,222] (ssl_manager:_/_)
         ssl_certificate.beam:254 (ssl_pkix_db:_/_)
         ssl_cipher.beam:[222,226,230,232,295] (crypto:block_decrypt/4)
         ssl_cipher.beam:[130,134,138,140,160,167] (crypto:block_encrypt/4)
         ssl_cipher.beam:205 (crypto:stream_decrypt/2)
         ssl_cipher.beam:126 (crypto:stream_encrypt/2)
         ssl_cipher.beam:104 (crypto:stream_init/2)
         ssl_cipher.beam:[1388,1405] (public_key:_/_)
         ssl_cipher.beam:[1394,1862,1863,1874] (ssl_certificate:_/_)
         ssl_cipher.beam:1437 (ssl_cipher:_/_)
         ssl_cipher.beam:312 (ssl_v3:_/_)
         ssl_cipher.beam:314 (tls_v1:_/_)
         ssl_config.beam:[108,118,123,128,143,153] (public_key:_/_)
         ssl_config.beam:[79,88] (ssl_certificate:_/_)
         ssl_config.beam:[44,46,62,101,150] (ssl_manager:_/_)
         ssl_connection.beam:[1735,1763,1797,1808] (crypto:generate_key/2)
         ssl_connection.beam:[1266,1415,1439,1481,1744] (public_key:_/_)
         ssl_connection.beam:[394,412,518,521,529,545,601,617,639,674,700,937,1192,1226,1302,1420,1444,1466,1486,1509,1538,1552,1563,1573,1580,1589,1600,1612,1625,1642,1657,1674,1716,1781,2090,2439] (ssl:_/_)
         ssl_connection.beam:[872,1993,1997,2001,2340,2343,2355,2367] (ssl_alert:_/_)
         ssl_connection.beam:[284,582,765,1181,1241,1255,2103,2138] (ssl_cipher:_/_)
         ssl_connection.beam:[317,2411] (ssl_config:_/_)
         ssl_connection.beam:1084 (ssl_connection:_/_)
         ssl_connection.beam:[318,394,412,492,518,528,545,564,584,601,617,639,673,700,822,937,1192,1226,1246,1267,1287,1301,1340,1347,1352,1359,1369,1376,1383,1396,1420,1444,1466,1486,1509,1538,1563,1573,1580,1589,1612,1625,1642,1658,1660,1674,1705,1716,1737,1746,1764,1772,1781,2090,2115] (ssl_handshake:_/_)
         ssl_connection.beam:[2024,2065,2069,2398,2400] (ssl_manager:_/_)
         ssl_connection.beam:[2014,2022] (ssl_pkix_db:_/_)
         ssl_connection.beam:[399,417,440,724,922,1082,1417,1441,1463,1483,1506,1535,1656,1691,1724,1726,1728,1730,1832,1840,1864,1867] (ssl_record:_/_)
         ssl_connection.beam:302 (ssl_session:_/_)
         ssl_connection.beam:[151,791,1910,1926,2282,2291,2346,2350] (ssl_socket:_/_)
         ssl_connection.beam:1819 (ssl_srp_primes:_/_)
         ssl_crl.beam:[51,51,81,83] (public_key:_/_)
         ssl_crl.beam:[37,44] (ssl_certificate:_/_)
         ssl_crl.beam:[33,59] (ssl_pkix_db:_/_)
         ssl_crl_cache.beam:[71,84,141,149] (public_key:_/_)
         ssl_crl_cache.beam:[87,92,97,108] (ssl_manager:_/_)
         ssl_crl_cache.beam:[46,165] (ssl_pkix_db:_/_)
         ssl_crl_hash_dir.beam:[42,69,101] (public_key:_/_)
         ssl_dist_sup.beam:60 (ssl_sup:_/_)
         ssl_handshake.beam:[491,498,509] (crypto:compute_key/4)
         ssl_handshake.beam:[195,375,377,383,386,406,484,544,546,593,595,614,634,636,1213,1612,1631,1633,1635,1638,2192,2202,2209,2212,2222,2228,2245,2255] (public_key:_/_)
         ssl_handshake.beam:[153,165,405,1531,1549,1560] (ssl_certificate:_/_)
         ssl_handshake.beam:[213,763,763,839,839,977,977,1089,1093,1097,1144,1671,1672,1735,1743,1829,1830,1877,1877,1882,1882,1923,1923,1969,1969] (ssl_cipher:_/_)
         ssl_handshake.beam:2181 (ssl_crl:_/_)
         ssl_handshake.beam:1225 (ssl_pkix_db:_/_)
         ssl_handshake.beam:[104,338,707,717,1348,1356,1366,1378,1383,1385,1389,1393,1409,1431,1666,1668,1674,1717,1719,1729] (ssl_record:_/_)
         ssl_handshake.beam:1157 (ssl_session:_/_)
         ssl_handshake.beam:506 (ssl_srp_primes:_/_)
         ssl_handshake.beam:[1641,1646,1679,1688] (ssl_v3:_/_)
         ssl_handshake.beam:2137 (tls_record:_/_)
         ssl_handshake.beam:[573,590,822,1030,1173,1643,1648,1684,1691,1759,1933] (tls_v1:_/_)
         ssl_manager.beam:582 (ssl_cipher:_/_)
         ssl_manager.beam:[120,132,146,148,177,264,321,326,337,346,379,384,389,432,462,598,600,602,606,607,696,728] (ssl_pkix_db:_/_)
         ssl_manager.beam:[485,493] (ssl_session:_/_)
         ssl_pkix_db.beam:[145,161,169,177,315,318,361] (public_key:_/_)
         ssl_record.beam:[368,385,400,420,451,485] (ssl_cipher:_/_)
         ssl_session.beam:[76,88] (ssl_manager:_/_)
         ssl_socket.beam:[107,109] (ssl_listen_tracker_sup:_/_)
         ssl_tls_dist_proxy.beam:[215,221,226,276,292,319,326,330,331,338,344,347,352,361,364,368] (ssl:_/_)
         tls.beam:[50,59,69,83,102,112] (ssl:_/_)
         tls_connection.beam:[133,489] (ssl_alert:_/_)
         tls_connection.beam:[78,79,92,93,171,199,255,279,281,353,356,363,370,379,399,402,406,422,427,442,445,448,460,463,576,586,639,641,688,699,710] (ssl_connection:_/_)
         tls_connection.beam:[144,205,480,613,622,624] (ssl_handshake:_/_)
         tls_connection.beam:[482,486,626] (ssl_record:_/_)
         tls_connection.beam:[557,646] (ssl_socket:_/_)
         tls_connection.beam:[76,90] (tls_connection_sup:_/_)
         tls_connection.beam:[200,252,277,330,415,479,623] (tls_handshake:_/_)
         tls_connection.beam:[204,494,531,546] (tls_record:_/_)
         tls_handshake.beam:[113,178] (ssl_cipher:_/_)
         tls_handshake.beam:[58,59,65,67,112,167,169,169,171,179,219,235,241,247,262,270,275,289] (ssl_handshake:_/_)
         tls_handshake.beam:[57,73] (ssl_record:_/_)
         tls_handshake.beam:69 (ssl_session:_/_)
         tls_handshake.beam:217 (ssl_v2:_/_)
         tls_handshake.beam:[56,99,115,116,165] (tls_record:_/_)
         tls_record.beam:[68,70,156,159,170,173,196,198,216,219,221,393] (ssl_record:_/_)
         tls_record.beam:423 (ssl_v3:_/_)
         tls_record.beam:426 (tls_v1:_/_)
         tls_v1.beam:412 (pubkey_cert_records:_/_)
         client_server.beam:[45,63] (public_key:_/_)
         client_server.beam:[33,34,41,42,44,47,48,50,52,60,62,64,67] (ssl:_/_)
         beam_lib.beam:892 (crypto:block_decrypt/4)
         c.beam:[89,158,176,230] (compile:file/2)
         escript.beam:[204,323,331,339,656] (compile:forms/2)
         qlc_pt.beam:441 (compile:noenv_forms/2)
         merl.beam:325 (compile:noenv_forms/2)
         cover.beam:1523 (compile:file/2)
         cover.beam:1578 (compile:forms/2)
         make.beam:268 (compile:file/2)
     10: Dynamic creation of atoms can exhaust atom memory
         asn1ct.beam:[1895,1917] (file:consult/1)
         ct_config_plain.beam:29 (file:consult/1)
         ct_config_xml.beam:48 (xmerl_sax_parser:_/_)
         ct_cover.beam:135 (file:consult/1)
         ct_logs.beam:[1122,1819] (file:consult/1)
         ct_make.beam:[90,146] (file:consult/1)
         ct_netconfc.beam:1365 (xmerl:_/_)
         ct_netconfc.beam:1400 (xmerl_sax_parser:_/_)
         ct_release_test.beam:[400,816] (file:consult/1)
         ct_run.beam:3267 (file:consult/1)
         ct_snmp.beam:[320,743,759,774,790,805,820,835,851] (file:consult/1)
         ct_testspec.beam:335 (file:consult/1)
         ct_util.beam:[235,969,974] (file:consult/1)
         ct_webtool.beam:1138 (file:consult/1)
         test_server_ctrl.beam:[229,1262,5219] (file:consult/1)
         test_server_node.beam:161 (file:consult/1)
         test_server_sup.beam:[187,297,922] (file:consult/1)
         compile.beam:805 (file:consult/1)
         diameter_dbg.beam:146 (file:consult/1)
         edoc_data.beam:[111,511,522] (xmerl_lib:_/_)
         edoc_doclet.beam:[244,263,453] (xmerl:_/_)
         edoc_layout.beam:[94,1015,1040] (xmerl:_/_)
         edoc_wiki.beam:85 (xmerl_scan:_/_)
         otpsgml_layout.beam:[82,811,831] (xmerl:_/_)
         docgen_edoc_xml_cb.beam:[45,51] (xmerl:_/_)
         docgen_otp_specs.beam:37 (xmerl:_/_)
         docgen_xmerl_xml_cb.beam:[53,56] (xmerl_lib:_/_)
         eunit_lib.beam:511 (file:path_consult/2)
         ic_options.beam:334 (file:consult/1)
         httpd_sup.beam:144 (file:consult/1)
         inets.beam:240 (file:consult/1)
         hdlt.beam:41 (file:consult/1)
         inet_db.beam:276 (file:consult/1)
         net_adm.beam:50 (file:path_consult/2)
         megaco.beam:704 (file:consult/1)
         megaco_erl_dist_encoder.beam:[181,194,206,235,239,246,271] (erlang:binary_to_term/1)
         megaco_codec_transform.beam:95 (file:consult/1)
         observer_trace_wx.beam:1135 (file:consult/1)
         egd_font.beam:67 (erlang:binary_to_term/1)
         reltool_server.beam:1430 (file:consult/1)
         dbg.beam:286 (file:consult/1)
         msacc.beam:93 (file:consult/1)
         system_information.beam:[105,486,610] (file:consult/1)
         release_handler.beam:[493,2075] (file:consult/1)
         release_handler.beam:526 (file:path_consult/2)
         systools_make.beam:[1725,1737] (file:consult/1)
         target_system.beam:39 (file:consult/1)
         snmp.beam:666 (file:consult/1)
         ssh.beam:650 (file:consult/1)
         beam_lib.beam:1061 (file:path_script/2)
         c.beam:481 (file:path_eval/2)
         instrument.beam:[84,107] (file:consult/1)
         make.beam:[93,149] (file:consult/1)
         xref_parser.beam:283 (erlang:list_to_atom/1)
         sudoku_gui.beam:272 (file:consult/1)
         xmerl.beam:[163,169,180,183,206,209,265] (xmerl_lib:_/_)
         xmerl_eventp.beam:[163,174,209,218,242,251] (xmerl:_/_)
         xmerl_eventp.beam:[144,167,179,195,213,222,230,246,255,263,272,287,307,380,381,401,404,407] (xmerl_scan:_/_)
         xmerl_lib.beam:[470,472,474,476,479,481,483,485,488,490,492,494,498,500,502,504,507,509,511,513] (xmerl_ucs:_/_)
         xmerl_sax_parser.beam:[99,103] (xmerl_sax_parser_list:_/_)
         xmerl_scan.beam:[307,323,881,970,2456,2464,2473,2496,2503,2732,2990,3023,3063,3307,3415,3422,3742] (xmerl_lib:_/_)
         xmerl_scan.beam:[560,712] (xmerl_ucs:_/_)
         xmerl_scan.beam:2208 (xmerl_uri:_/_)
         xmerl_scan.beam:592 (xmerl_validate:_/_)
         xmerl_scan.beam:614 (xmerl_xsd:_/_)
         xmerl_simple.beam:[39,43,71,82,83,92,94,101,103,106,107] (xmerl_scan:_/_)
         xmerl_validate.beam:[234,300,319] (xmerl_lib:_/_)
         xmerl_xlate.beam:47 (xmerl:_/_)
         xmerl_xlate.beam:[42,46] (xmerl_scan:_/_)
         xmerl_xpath.beam:361 (xmerl_xpath_lib:_/_)
         xmerl_xpath.beam:225 (xmerl_xpath_parse:_/_)
         xmerl_xpath.beam:294 (xmerl_xpath_pred:_/_)
         xmerl_xpath.beam:224 (xmerl_xpath_scan:_/_)
         xmerl_xpath_lib.beam:[37,44] (xmerl_xpath_pred:_/_)
         xmerl_xpath_pred.beam:773 (xmerl_scan:_/_)
         xmerl_xpath_pred.beam:[136,292] (xmerl_xpath:_/_)
         xmerl_xpath_pred.beam:[790,802] (xmerl_xpath_scan:_/_)
         xmerl_xpath_scan.beam:[216,237,249] (xmerl_lib:_/_)
         xmerl_xs.beam:[97,100,118] (xmerl_lib:_/_)
         xmerl_xs.beam:109 (xmerl_xpath:_/_)
         xmerl_xsd.beam:[235,301,303,343,345,1951,1959] (xmerl_scan:_/_)
         xmerl_xsd.beam:[3393,3425] (xmerl_xpath:_/_)
         xmerl_xsd.beam:[184,191] (xmerl_xsd:_/_)
         xmerl_xsd.beam:[3206,5132] (xmerl_xsd_type:_/_)
         xmerl_xsd_type.beam:554 (xmerl_b64Bin:_/_)
         xmerl_xsd_type.beam:554 (xmerl_b64Bin_scan:_/_)
         xmerl_xsd_type.beam:[55,62,143,610,615,624,628,634,638,645] (xmerl_lib:_/_)
         xmerl_xsd_type.beam:[781,784] (xmerl_regexp:_/_)
         xmerl_xsd_type.beam:130 (xmerl_uri:_/_)
         xmerl_xsd_type.beam:[809,813,852,862,871,898,908,917,942,952,961,986,996,1005] (xmerl_xsd_type:_/_)

Module usage like the xmerl application using xmerl modules is redundant, but
the output may be used to understand how Erlang source code could have
security problems that are not reported by the pest.erl script when it is ran
on Erlang source code, due to an indirect function call.

If you want to include indirect function calls in the security scan pest.erl
performs, add each dependency that needs to be included in the scan with the
`-d` command line argument.  If you need to examine the resulting checks
after the dependencies are processed, but before the scan, add the `-i`
command line argument to display all the checks.  Below is the
resulting expanded checks after including Erlang/OTP indirect function calls
(7441 total lines that represent an exhaustive search for possible problems):

    $ ./pest.erl -v -b -d ~/installed/lib/erlang/lib/ -i
    [{90,"Port Drivers may cause undefined behavior",
      [{crashdump_viewer,debug,1},
       {ct_webtool,debug,1},
       {ct_webtool,debug_app,1},
       {dbg,trace_port,2},
       {dbg,trace_port1,3},
       {diameter_dbg,tracer,1},
       {erl_ddll,do_load_driver,3},
       {erl_ddll,load,2},
       {erl_ddll,load_driver,2},
       {erl_ddll,reload,2},
       {erl_ddll,reload_driver,2},
       {erl_ddll,try_load,3},
       {et_collector,monitor_trace_port,2},
       {et_collector,start_trace_port,1},
       {etop,start,0},
       {etop,start,1},
       {etop_tr,setup_tracer,1},
       {fprof,'$code_change',1},
       {fprof,analyse,0},
       {fprof,analyse,1},
       {fprof,analyse,2},
       {fprof,apply,2},
       {fprof,apply,3},
       {fprof,apply,4},
       {fprof,apply_1,3},
       {fprof,apply_1,4},
       {fprof,apply_continue,4},
       {fprof,apply_start_stop,4},
       {fprof,call,1},
       {fprof,handle_req,3},
       {fprof,load_profile,0},
       {fprof,load_profile,1},
       {fprof,load_profile,2},
       {fprof,open_dbg_trace_port,2},
       {fprof,profile,0},
       {fprof,profile,1},
       {fprof,profile,2},
       {fprof,save_profile,0},
       {fprof,save_profile,1},
       {fprof,save_profile,2},
       {fprof,server_loop,1},
       {fprof,start,0},
       {fprof,trace,1},
       {fprof,trace,2},
       {inets,enable_trace,2},
       {inets,enable_trace,3},
       {inets_trace,enable,2},
       {inets_trace,enable,3},
       {inets_trace,enable2,3},
       {megaco,enable_trace,2},
       {megaco_codec_meas,flex_scanner_handler,1},
       {megaco_codec_mstone_lib,flex_scanner_handler,1},
       {megaco_flex_scanner,do_start,1},
       {megaco_flex_scanner,load_driver,1},
       {megaco_flex_scanner,start,0},
       {megaco_flex_scanner,start,1},
       {megaco_flex_scanner_handler,bump_flex_scanner,1},
       {megaco_flex_scanner_handler,code_change,3},
       {megaco_flex_scanner_handler,init,1},
       {megaco_flex_scanner_handler,start_flex_scanners,0},
       {megaco_simple_mg,init_batch,4},
       {megaco_simple_mg,init_inline_trace,1},
       {megaco_simple_mg,start,0},
       {megaco_simple_mg,start,3},
       {megaco_simple_mgc,init_batch,3},
       {megaco_simple_mgc,init_inline_trace,1},
       {megaco_simple_mgc,start,0},
       {megaco_simple_mgc,start,2},
       {megaco_simple_mgc,start,4},
       {observer_trace_wx,handle_event,2},
       {percept,profile,1},
       {percept,profile,2},
       {percept,profile,3},
       {percept_profile,profile_to_file,2},
       {percept_profile,start,1},
       {percept_profile,start,2},
       {percept_profile,start,3},
       {test_server_node,add_nodes,3},
       {test_server_node,trc,1},
       {test_server_node,trc_loop,3},
       {ttb,do_tracer,3},
       {ttb,do_tracer,4},
       {ttb,ip_to_file,2},
       {ttb,start_trace,4},
       {ttb,tracer,0},
       {ttb,tracer,1},
       {ttb,tracer,2},
       {wxe_master,init,1}]},
     {90,"NIFs may cause undefined behavior",
      [{asn1rt_nif,load_nif,0},
       {crypto,on_load,0},
       {dyntrace,on_load,0},
       {erl_tracer,on_load,0},
       {erlang,load_nif,2},
       {init,boot,1}]},
     {80,"OS shell usage may require input validation",
      [{cpu_sup,get_uint32_measurement,2},
    ...7344 more lines...                ]}]

For comparison, the default checks specified in the pest.erl source code
are below (92 lines that represent [all core problems](https://github.com/okeuday/pest/blob/master/src/pest.erl#L153-L215)):

    $ ./pest.erl -v -i
    [{90,"Port Drivers may cause undefined behavior",
      [{erl_ddll,load,2},
       {erl_ddll,load_driver,2},
       {erl_ddll,reload,2},
       {erl_ddll,reload_driver,2},
       {erl_ddll,try_load,3}]},
     {90,"NIFs may cause undefined behavior",[{erlang,load_nif,2}]},
     {80,"OS shell usage may require input validation",[{os,cmd,1}]},
     {80,"OS process creation may require input validation",
      [{erlang,open_port,2}]},
     {15,"Keep OpenSSL updated for crypto module use (run with \"-V crypto\")",
      ['OTP-PUB-KEY','PKCS-FRAME',dtls,dtls_connection,dtls_connection_sup,
       dtls_handshake,dtls_record,dtls_v1,inet6_tls_dist,inet_tls_dist,
       pubkey_cert,pubkey_cert_records,pubkey_crl,pubkey_pbe,pubkey_pem,
       pubkey_ssh,public_key,snmp,snmp_app,snmp_app_sup,snmp_community_mib,
       snmp_conf,snmp_config,snmp_framework_mib,snmp_generic,snmp_generic_mnesia,
       snmp_index,snmp_log,snmp_mini_mib,snmp_misc,snmp_note_store,
       snmp_notification_mib,snmp_pdus,snmp_shadow_table,snmp_standard_mib,
       snmp_target_mib,snmp_user_based_sm_mib,snmp_usm,snmp_verbosity,
       snmp_view_based_acm_mib,snmpa,snmpa_acm,snmpa_agent,snmpa_agent_sup,
       snmpa_app,snmpa_authentication_service,snmpa_conf,snmpa_discovery_handler,
       snmpa_discovery_handler_default,snmpa_error,snmpa_error_io,
       snmpa_error_logger,snmpa_error_report,snmpa_local_db,snmpa_mib,
       snmpa_mib_data,snmpa_mib_data_tttn,snmpa_mib_lib,snmpa_mib_storage,
       snmpa_mib_storage_dets,snmpa_mib_storage_ets,snmpa_mib_storage_mnesia,
       snmpa_misc_sup,snmpa_mpd,snmpa_net_if,snmpa_net_if_filter,
       snmpa_network_interface,snmpa_network_interface_filter,
       snmpa_notification_delivery_info_receiver,snmpa_notification_filter,
       snmpa_set,snmpa_set_lib,snmpa_set_mechanism,snmpa_supervisor,snmpa_svbl,
       snmpa_symbolic_store,snmpa_target_cache,snmpa_trap,snmpa_usm,snmpa_vacm,
       snmpc,snmpc_lib,snmpc_mib_gram,snmpc_mib_to_hrl,snmpc_misc,snmpc_tok,snmpm,
       snmpm_conf,snmpm_config,snmpm_misc_sup,snmpm_mpd,snmpm_net_if,
       snmpm_net_if_filter,snmpm_net_if_mt,snmpm_network_interface,
       snmpm_network_interface_filter,snmpm_server,snmpm_server_sup,
       snmpm_supervisor,snmpm_user,snmpm_user_default,snmpm_user_old,snmpm_usm,
       ssh,ssh_acceptor,ssh_acceptor_sup,ssh_app,ssh_auth,ssh_bits,ssh_channel,
       ssh_channel_sup,ssh_cli,ssh_client_key_api,ssh_connection,
       ssh_connection_handler,ssh_connection_sup,ssh_daemon_channel,ssh_dbg,
       ssh_file,ssh_info,ssh_io,ssh_message,ssh_no_io,ssh_server_key_api,ssh_sftp,
       ssh_sftpd,ssh_sftpd_file,ssh_sftpd_file_api,ssh_shell,ssh_subsystem_sup,
       ssh_sup,ssh_system_sup,ssh_transport,ssh_xfer,sshc_sup,sshd_sup,ssl,
       ssl_alert,ssl_app,ssl_certificate,ssl_cipher,ssl_config,ssl_connection,
       ssl_crl,ssl_crl_cache,ssl_crl_cache_api,ssl_crl_hash_dir,ssl_dist_sup,
       ssl_handshake,ssl_listen_tracker_sup,ssl_manager,ssl_pkix_db,ssl_record,
       ssl_session,ssl_session_cache,ssl_session_cache_api,ssl_socket,
       ssl_srp_primes,ssl_sup,ssl_tls_dist_proxy,ssl_v2,ssl_v3,tls,tls_connection,
       tls_connection_sup,tls_handshake,tls_record,tls_v1,
       {compile,file,2},
       {compile,forms,2},
       {compile,noenv_file,2},
       {compile,noenv_forms,2},
       {crypto,block_decrypt,3},
       {crypto,block_decrypt,4},
       {crypto,block_encrypt,3},
       {crypto,block_encrypt,4},
       {crypto,compute_key,4},
       {crypto,ec_curve,1},
       {crypto,generate_key,2},
       {crypto,generate_key,3},
       {crypto,next_iv,2},
       {crypto,next_iv,3},
       {crypto,private_decrypt,4},
       {crypto,private_encrypt,4},
       {crypto,public_decrypt,4},
       {crypto,public_encrypt,4},
       {crypto,sign,4},
       {crypto,stream_decrypt,2},
       {crypto,stream_encrypt,2},
       {crypto,stream_init,2},
       {crypto,stream_init,3},
       {crypto,verify,5}]},
     {10,"Dynamic creation of atoms can exhaust atom memory",
      [xmerl,xmerl_b64Bin,xmerl_b64Bin_scan,xmerl_eventp,xmerl_html,xmerl_lib,
       xmerl_otpsgml,xmerl_regexp,xmerl_sax_old_dom,xmerl_sax_parser,
       xmerl_sax_parser_latin1,xmerl_sax_parser_list,xmerl_sax_parser_utf16be,
       xmerl_sax_parser_utf16le,xmerl_sax_parser_utf8,xmerl_sax_simple_dom,
       xmerl_scan,xmerl_sgml,xmerl_simple,xmerl_text,xmerl_ucs,xmerl_uri,
       xmerl_validate,xmerl_xlate,xmerl_xml,xmerl_xpath,xmerl_xpath_lib,
       xmerl_xpath_parse,xmerl_xpath_pred,xmerl_xpath_scan,xmerl_xs,xmerl_xsd,
       xmerl_xsd_type,
       {erlang,binary_to_atom,2},
       {erlang,binary_to_term,1},
       {erlang,list_to_atom,1},
       {file,consult,1},
       {file,eval,1},
       {file,eval,2},
       {file,path_consult,2},
       {file,path_eval,2},
       {file,path_script,2},
       {file,path_script,3},
       {file,script,1},
       {file,script,2}]}]

See [Usage](https://github.com/okeuday/pest/#usage) for more information.

Limitations
-----------

* All function calls that are checked use an Erlang atom for the module name
  and function name, so if a variable is used for either, it is possible that
  the function usage will not be seen by PEST.  If you are concerned about
  this problem, you can make sure optimizations are being used during
  compilation and confirm you are using PEST with the beam output of the
  compilation.

Author
------

Michael Truog (mjtruog [at] gmail (dot) com)

License
-------

MIT License

