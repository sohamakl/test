# Highligter

##

```sql
<?php

namespace Plugins\Highlight;

use \Typemill\Plugin;

class Highlight extends Plugin
{   
    public static function getSubscribedEvents()
    {
        return array(
            'onTwigLoaded'          => 'onTwigLoaded'
        );
    }

    public function onTwigLoaded()
    {
        # only add to frontend
        if(!$this->adminroute OR strpos($this->route, 'ebooks/preview') !== false OR strpos($this->route, 'kixotepdf') !== false)
        {
            $highlightSettings = $this->getPluginSettings();

            /* add external CSS and JavaScript */
            $this->addCSS('/highlight/public/themes/reset.css');

            $dayTheme   = isset($highlightSettings['theme']) ? $highlightSettings['theme'] : 'default';
            $nightTheme = isset($highlightSettings['nightTheme']) ? $highlightSettings['nightTheme'] : 'default';

            // Add the initial theme (defaults to day theme)
            $this->addCSS('/highlight/public/themes/'.$dayTheme.'.css');

            $this->addJS('/highlight/public/js/highlight.min.js');
            $this->addJS('/highlight/public/js/highlightjs-line-numbers.min.js');

            if (isset($highlightSettings['copyButton']) && $highlightSettings['copyButton'] == 'true')
            {
                /* initialize copy badge 
                https://github.com/arronhunt/highlightjs-copy
                */
                $this->addCss('/highlight/public/js/highlightjs-copy.min.css');
                $this->addJS('/highlight/public/js/highlightjs-copy.min.js');
            }

            /* initialize the script */
            $this->addInlineJS('
                setTimeout(function() {
                    ' . ((isset($highlightSettings['copyButton']) && $highlightSettings['copyButton'] == 'true') ? 'hljs.addPlugin(new CopyButtonPlugin());' : '') . '
                    hljs.highlightAll();
                    ' . ((isset($highlightSettings['lineNumber']) && $highlightSettings['lineNumber'] == 'true') || !isset($highlightSettings['lineNumber']) ? 'hljs.initLineNumbersOnLoad({ singleLine: true });' : '') . '
                }, 500);
            ');

            /* Add theme switcher script */
            $this->addInlineJS('
                (function() {
                    var dayTheme = "'.$dayTheme.'";
                    var nightTheme = "'.$nightTheme.'";

                    function getThemeLink() {
                        var links = document.getElementsByTagName("link");
                        for (var i = 0; i < links.length; i++) {
                            var href = links[i].getAttribute("href");
                            if (href && href.indexOf("/highlight/public/themes/") > -1 && href.indexOf("reset.css") === -1) {
                                return links[i];
                            }
                        }
                        return null;
                    }

                    function updateTheme(theme) {
                        var link = getThemeLink();
                        if (link) {
                            var targetTheme = (theme === "dark") ? nightTheme : dayTheme;
                            var currentHref = link.getAttribute("href");

                            // Replace the theme filename in the existing path to preserve base URL
                            // Matches "/themes/[anything].css"
                            var newHref = currentHref.replace(/\/themes\/[^\/]+\.css/, "/themes/" + targetTheme + ".css");

                            if (currentHref !== newHref) {
                                link.setAttribute("href", newHref);
                            }
                        }
                    }

                    var observer = new MutationObserver(function(mutations) {
                        mutations.forEach(function(mutation) {
                            if (mutation.type == "attributes" && mutation.attributeName == "data-theme") {
                                var theme = document.documentElement.getAttribute("data-theme");
                                updateTheme(theme);
                            }
                        });
                    });

                    observer.observe(document.documentElement, {
                        attributes: true
                    });

                    // Initial check
                    var currentTheme = document.documentElement.getAttribute("data-theme");
                    if(currentTheme === "dark") {
                        updateTheme("dark");
                    }
                })();
            ');

            if (isset($highlightSettings['wordWrap']) && $highlightSettings['wordWrap'] == 'true')
            {
                $this->addInlineCSS('
                    code.hljs {
                          white-space: pre-wrap;
                        }
                    ');
            }

            if (isset($highlightSettings['displayLanguage']) && $highlightSettings['displayLanguage'] == 'true')
            {
                $this->addInlineCSS('
                    pre code[data-language]::before {
                        content: attr(data-language);
                        position: absolute;
                        top: 0;
                        left: 0;
                        padding: 4px 8px;
                        color: #333;
                        background: #f4f4f4;
                        font-size: 0.75rem;
                        border-bottom-right-radius: 4px;
                        z-index: 10;
                    }
                    pre code[data-language] {
                        padding-top: 30px !important;
                    }
                    pre {
                        position: relative;
                    }
                    /* PDF/No-JS Fallback for Styling */
                    pre {
                        background: transparent !important;
                        padding: 0 !important;
                        border: none !important;
                        margin: 0 !important;
                    }
                    pre code {
                        display: block;
                        overflow-x: auto;
                        padding: 1em;
                        background: #F3F3F3;
                        color: #444;
                        border-radius: 4px;
                    }

                ');

                $this->addInlineJS('
                    setTimeout(function() {
                        document.querySelectorAll("code[class^=\'language-\'], code[class*=\' language-\']").forEach(function(code) {
                            var match = code.className.match(/language-([a-z0-9]+)/);
                            if (match && match[1]) {
                                code.setAttribute("data-language", match[1]);
                            }
                        });
                    }, 600);
                ');
            }

        }
    }
}
```

