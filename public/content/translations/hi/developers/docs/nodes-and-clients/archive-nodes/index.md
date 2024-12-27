---
title: एथेरियम आर्काइव नोड
description: आर्काइव नोड्स का अवलोकन
lang: hi
sidebarDepth: 2
---

एक आर्काइव नोड एक एथेरियम क्लाइंट का एक उदाहरण है जिसे सभी ऐतिहासिक स्थितियों का आर्काइव बनाने के लिए कॉन्फ़िगर किया गया है। यह कुछ उपयोग के मामलों के लिए एक उपयोगी उपकरण है लेकिन पूर्ण नोड की तुलना में चलाना अधिक मुश्किल हो सकता है।

## आवश्यक शर्तें {#prerequisites}

आपको [एथेरियम नोड](/developers/docs/nodes-and-clients/) की अवधारणा, इसकी [वास्तुकला](/developers/docs/nodes-and-clients/node-architecture/), [सिंक रणनीतियों](/developers/docs/nodes-and-clients/#sync-modes), उन्हें [चलाने](/developers/docs/nodes-and-clients/run-a-node/) और [उपयोग करने](/developers/docs/apis/json-rpc/) की प्रथाओं को समझना चाहिए।

## एक आर्काइव नोड क्या है

एक आर्काइव नोड के महत्व को समझने के लिए, आइए "स्थिति" की अवधारणा को स्पष्ट करें। एथेरियम को _लेनदेन-आधारित स्थिति मशीन_ के रूप में संदर्भित किया जा सकता है। इसमें लेनदेन को निष्पादित करने वाले खाते और एप्लिकेशन शामिल हैं जो अपनी स्थिति बदल रहे हैं। प्रत्येक खाते और अनुबंध के बारे में जानकारी के साथ वैश्विक डेटा को स्टेट नामक एक ट्राई डेटाबेस में संग्रहित किया जाता है। इसे निष्पादन परत (EL) क्लाइंट द्वारा नियंत्रित किया जाता है और इसमें शामिल हैं:

- खाता शेष और नॉन्सेस
- अनुबंध कोड और भंडारण
- सहमति से संबंधित डेटा, जैसे स्टेकिंग डिपॉजिट अनुबंध

नेटवर्क के साथ बातचीत करने, नए ब्लॉकों को सत्यापित करने और उत्पादन करने के लिए, एथेरियम क्लाइंट को सबसे हाल के परिवर्तनों (श्रृंखला की नोक) और इसलिए वर्तमान स्थिति के साथ रहना होगा। एक पूर्ण नोड के रूप में कॉन्फ़िगर किया गया एक निष्पादन परत क्लाइंट नेटवर्क की नवीनतम स्थिति को सत्यापित करता है और उसका अनुसरण करता है, लेकिन केवल पिछली कुछ स्थिति को कैश करता है, उदाहरण के लिए अंतिम 128 ब्लॉकों से जुड़ी स्थिति, इसलिए यह चेन रीऑर्ग को संभाल सकता है और हाल के डेटा तक तेजी से पहुंच प्रदान कर सकता है। हाल की स्थिति वह है जो सभी क्लाइंट को आने वाले लेनदेन को सत्यापित करने और नेटवर्क का उपयोग करने की आवश्यकता है।

आप किसी दिए गए ब्लॉक और इतिहास रीप्ले के रूप में आर्काइव पर एक क्षणिक नेटवर्क स्नैपशॉट के रूप में स्थिति की कल्पना कर सकते हैं।

ऐतिहासिक states को सुरक्षित रूप से छंटनी की जा सकती है क्योंकि वे नेटवर्क को संचालित करने के लिए आवश्यक नहीं हैं और क्लाइंट के लिए सभी आउट-ऑफ-डेट डेटा रखना बेकार होगा। कुछ हालिया ब्लॉक (जैसे हेड से पहले 128 ब्लॉक) से पहले मौजूद स्थितियों को प्रभावी ढंग से फेंक दिया जाता है। पूर्ण नोड्स केवल ऐतिहासिक ब्लॉकचेन डेटा (ब्लॉक और लेनदेन) और कभी-कभी ऐतिहासिक स्नैपशॉट रखते हैं जिनका उपयोग वे अनुरोध पर पुरानी स्थितियों को पुनः उत्पन्न करने के लिए कर सकते हैं। वे EVM में पिछले लेनदेन को फिर से निष्पादित करके ऐसा करते हैं, जिसकी कम्प्यूटेशनल रूप से तब मांग की जा सकती है जब वांछित स्थिति निकटतम स्नैपशॉट से दूर हो।

हालांकि, इसका मतलब यह है कि एक पूर्ण नोड पर एक ऐतिहासिक स्थिति तक पहुंचने में बहुत अधिक गणना की खपत होती है। क्लाइंट को पिछले सभी लेनदेन निष्पादित करने और उत्पत्ति से एक ऐतिहासिक स्थिति की गणना करने की आवश्यकता हो सकती है। आर्काइव नोड्स न केवल सबसे हाल की स्थितियों बल्कि प्रत्येक ब्लॉक के बाद बनाई गई प्रत्येक ऐतिहासिक स्थिति को संग्रहित करके इसे हल करते हैं। यह मूल रूप से बड़ी डिस्क स्थान की आवश्यकता के साथ एक ट्रेड-ऑफ़ करता है।

यह ध्यान रखना महत्वपूर्ण है कि नेटवर्क सभी ऐतिहासिक डेटा को रखने और प्रदान करने के लिए आर्काइव नोड्स पर निर्भर नहीं करता है। जैसा कि ऊपर उल्लेख किया गया है, सभी ऐतिहासिक अंतरिम स्थितियों को एक पूर्ण नोड पर प्राप्त किया जा सकता है। लेन-देन किसी भी पूर्ण नोड (वर्तमान में 400G से कम) द्वारा संग्रहित किए जाते हैं और पूरे आर्काइव को बनाने के लिए फिर से चलाए जा सकते हैं।

### उपयोग के मामले

एथेरियम का नियमित उपयोग जैसे लेनदेन भेजना, अनुबंधों को परिनियोजित करना, सहमति की पुष्टि करना आदि के लिए ऐतिहासिक स्थितियों तक पहुंच की आवश्यकता नहीं होती है। यूज़र को नेटवर्क के साथ मानक इंटरैक्शन के लिए आर्काइव नोड की आवश्यकता नहीं होती है।

स्थिति आर्काइव का मुख्य लाभ ऐतिहासिक स्थितियों के बारे में प्रश्नों तक त्वरित पहुंच है। उदाहरण के लिए, आर्काइव नोड तुरंत परिणाम लौटाएगा जैसे:

- _खाता 0x1337 का... ब्लॉक 15537393 में ETH बैलेंस क्या था?_
- _ब्लॉक 1920000 पर अनुबंध 0x में टोकन 0x का बैलेंस क्या है?_

जैसा कि ऊपर बताया गया है, एक पूर्ण नोड को EVM निष्पादन द्वारा इस डेटा को उत्पन्न करने की आवश्यकता होगी जो CPU का उपयोग करता है और समय लेता है। आर्काइव नोड्स उन्हें डिस्क पर एक्सेस करते हैं और तुरंत प्रतिक्रियाएं देते हैं। यह बुनियादी ढांचे के कुछ हिस्सों के लिए एक उपयोगी विशेषता है, उदाहरण के लिए:

- ब्लॉक खोजकर्ता जैसे सेवा प्रदाता
- शोधकर्ताओं
- सुरक्षा विश्लेषक
- Dapp डेवलपर
- ऑडिटिंग और अनुपालन

विभिन्न मुफ्त [सेवाएं](/developers/docs/nodes-and-clients/nodes-as-a-service/) हैं जो ऐतिहासिक डेटा तक पहुंच की अनुमति भी देती हैं। चूंकि एक आर्काइव नोड चलाने की अधिक मांग है, यह पहुंच ज्यादातर सीमित है और केवल सामयिक पहुंच के लिए काम करती है। यदि आपकी परियोजना को ऐतिहासिक डेटा तक निरंतर पहुंच की आवश्यकता है, तो आपको स्वयं एक चलाने पर विचार करना चाहिए।

## कार्यान्वयन और उपयोग

इस संदर्भ में आर्काइव नोड का अर्थ है यूज़र द्वारा निष्पादन परत क्लाइंट द्वारा प्रदान किया गया डेटा क्योंकि वे स्थिति डेटाबेस को संभालते हैं और JSON-RPC समापन बिंदु प्रदान करते हैं। कॉन्फ़िगरेशन विकल्प, सिंक्रनाइज़ेशन समय और डेटाबेस आकार क्लाइंट के अनुसार भिन्न हो सकते हैं। विवरण के लिए, कृपया अपने क्लाइंट द्वारा प्रदान किए गए प्रलेखन देखें।

अपना स्वयं का आर्काइव नोड शुरू करने से पहले, क्लाइंट और विशेष रूप से विभिन्न [हार्डवेयर आवश्यकताओं](/developers/docs/nodes-and-clients/run-a-node/#requirements) के बीच अंतर के बारे में जानें। अधिकांश क्लाइंट इस सुविधा के लिए अनुकूलित नहीं हैं और उनके आर्काइव के लिए 12TB से अधिक स्थान की आवश्यकता होती है। इसके विपरीत, Erigon जैसे कार्यान्वयन एक ही डेटा को 3TB से कम में संग्रहित कर सकते हैं जो उन्हें आर्काइव नोड चलाने का सबसे प्रभावी तरीका बनाता है।

## अनुशंसित अभ्यास

[नोड चलाने के लिए सामान्य सिफारिशों](/developers/docs/nodes-and-clients/run-a-node/) के अलावा, एक आर्काइव नोड हार्डवेयर और रखरखाव पर अधिक मांग कर सकता है। Erigons की [प्रमुख विशेषताओं](https://github.com/ledgerwatch/erigon#key-features) को ध्यान में रखते हुए सबसे व्यावहारिक दृष्टिकोण [Erigon](/developers/docs/nodes-and-clients/#erigon) क्लाइंट कार्यान्वयन का उपयोग कर रहा है।

### हार्डवेयर

क्लाइंट के प्रलेखन में किसी दिए गए मोड के लिए हार्डवेयर आवश्यकताओं को हमेशा सत्यापित करना सुनिश्चित करें। आर्काइव नोड्स के लिए सबसे बड़ी आवश्यकता डिस्क स्थान है। क्लाइंट के आधार पर, यह 3TB से 12TB तक भिन्न होता है। यहां तक कि अगर HDD को बड़ी मात्रा में डेटा के लिए बेहतर समाधान माना जा सकता है, तो इसे सिंक करने और श्रृंखला के प्रमुख को लगातार अपडेट करने के लिए SSD ड्राइव की आवश्यकता होगी। [SATA](https://www.cleverfiles.com/help/sata-hard-drive.html) ड्राइव काफी अच्छे हैं लेकिन यह एक विश्वसनीय गुणवत्ता होनी चाहिए, कम से कम [TLC](https://blog.synology.com/tlc-vs-qlc-ssds-what-are-the-differences)। डिस्क को डेस्कटॉप कंप्यूटर या पर्याप्त स्लॉट वाले सर्वर में फिट किया जा सकता है। ऐसे समर्पित उपकरण उच्च अपटाइम नोड चलाने के लिए आदर्श हैं। इसे लैपटॉप पर चलाना पूरी तरह से संभव है लेकिन पोर्टेबिलिटी के लिए अतिरिक्त लागत आएगी।

सभी डेटा को एक वॉल्यूम में फिट करने की आवश्यकता है, इसलिए डिस्क को जोड़ना होगा, उदाहरण के लिए [RAID0](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_0) या [LVM](https://web.mit.edu/rhel-doc/5/RHEL-5-manual/Deployment_Guide-en-US/ch-lvm.html) के साथ। यह [ZFS](https://en.wikipedia.org/wiki/ZFS) का उपयोग करने पर विचार करने के लायक भी हो सकता है क्योंकि यह "कॉपी-ऑन-राइट" का सपोर्ट करता है जो सुनिश्चित करता है कि डेटा बिना किसी निम्न स्तर की त्रुटियों के डिस्क पर सही ढंग से लिखा गया है।

अचानक डेटाबेस खराब होने को रोकने में अधिक स्थिरता और सुरक्षा के लिए, विशेष रूप से एक पेशेवर सेटअप में, [ECC मेमोरी](https://en.wikipedia.org/wiki/ECC_memory) का उपयोग करने पर विचार करें यदि आपका सिस्टम इसका समर्थन करता है। RAM का आकार आमतौर पर एक पूर्ण नोड के समान होने की सलाह दी जाती है, लेकिन अधिक RAM सिंक्रनाइज़ेशन को गति देने में मदद कर सकता है।

प्रारंभिक सिंक के दौरान, आर्काइव मोड में क्लाइंट उत्पत्ति के बाद से प्रत्येक लेनदेन निष्पादित करेंगे। निष्पादन की गति ज्यादातर CPU द्वारा सीमित होती है, इसलिए एक तेज CPU प्रारंभिक सिंक समय में मदद कर सकता है। एक औसत उपभोक्ता कंप्यूटर पर, प्रारंभिक सिंक में एक महीने तक का समय लग सकता है।

## अग्रिम पठन {#further-reading}

- [एथेरियम फुल नोड बनाम आर्काइव नोड](https://www.quicknode.com/guides/infrastructure/ethereum-full-node-vs-archive-node) - _QuickNode, सितंबर 2022_
- [अपनी खुद की एथेरियम आर्काइव नोड का निर्माण](https://tjayrush.medium.com/building-your-own-ethereum-archive-node-72c014affc09) - _थॉमस जे रश, अगस्त 2021_
- [Erigon, Erigon का RPC और TrueBlocks (स्क्रैप और API) को सेवाओं के रूप में कैसे सेट करें](https://magnushansson.xyz/blog_posts/crypto_defi/2022-01-10-Erigon-Trueblocks) _– मैग्नस हैनसन, सितंबर 2022 को अपडेट किया गया_

## संबंधित विषय {#related-topics}

- [ नोड्स और क्लाइंट](/developers/docs/nodes-and-clients/)
- [नोड चलाना](/developers/docs/nodes-and-clients/run-a-node/)