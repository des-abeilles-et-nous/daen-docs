# Data pipelines High Level Overview

```mermaid
flowchart BT
    mnhn[(MNHN)]
	siren[(SIREN)]
    ana[(analytics)]
	log[(logs)]
	app(Beefree app)
	bq[[GCP/BigQuery]]
    fb[/Facebook/]
    ggl[/Google/]
    apl[/Apple/]
    lkr[\Looker/]
    firebase[(Firebase)]

    subgraph "App logic"
		firebase <==POI, User (native sdk) ==> app
		stry[/sentry/] -.debug (!).-> log
		app --debug (sdk auto)--> stry
        app --Usage, debug (sdk)--> log
	end
    mnhn --POI.hornet.asian (sched scraping) --> firebase
    siren --POI.apiary,POS,Opportunity (static) --> app
    firebase --POI (sched addon) --> bq
    log --Usage(GCP sink)--> bq
    app --Audience, Usage (auto)--> ana
    fb -.Audience, Usage (!).-> ana
    app <--User (OAuth)---> oa
    subgraph oa [OAuth]
        direction LR
	fb
    	apl
        ggl
    end
    app --Usage (sdk)--> fb
    bq --> lkr
    ana --> lkr
    ana --> ga[\adhoc Analytics/]
```

