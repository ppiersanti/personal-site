{:title "shadows-cljs + Cider"
 :layout :post
 :tags  ["Clojure" "clojure" "clojurescript" "shadows-cljs" "cider"]
 :toc true
 :draft? false}

## Shadows-cljs + Cider

The following Shadows-cljs project configuration enable to launch a REPL directly from Cider,
opposed to the default setup, which starts a REPL from Shadows-cljs to which you can
connect from Cider, choosing `Connect to an existing REPL`.

```clojure
{:source-paths
 ["src/cljs"]

 :dependencies [[binaryage/devtools "0.9.7"]
				[reagent "0.8.0-alpha2"]
				[cider/cider-nrepl "0.22.2"]
				[cider/piggieback "0.4.1"]
				[refactor-nrepl "2.5.0-SNAPSHOT"]]

 ;; set an nrepl port for connection to a REPL.
 :nrepl {:port       8777
		 :middleware [refactor-nrepl.middleware/wrap-refactor]}

 :builds
 {:app {:target     :browser
		:output-dir "public/js/compiled"
		:asset-path "/js/compiled"

		:modules
		{:main
		 {:entries [your.ns]}}

		:devtools
		;; before live-reloading any code call this function
		{:before-load vg.core/stop
		 ;; after live-reloading finishes call this function
		 :after-load  your.ns/start
		 ;; serve the public directory over http at port 8700
		 :http-root   "public"
		 :http-port   8700
		 :preloads    [devtools.preload]}
		}}}
```
