# H5P-Quiz-with-attempts
Quiz with attempts but for now we have to also edit moodle core files

Clone this repositery then use :
```
zip -r -D -X pimenkoquestionset.h5p *
```
To made the H5P package.


# Copy and paste this code in h5p/classes/player.php replace display function
```
    /**
     * Get the encoded URL for embeding this H5P content.
     *
     * @param string $url Local URL of the H5P file to display.
     * @param stdClass $config Configuration for H5P buttons.
     * @param bool $preventredirect Set to true in scripts that can not redirect (CLI, RSS feeds, etc.), throws exceptions
     * @param string $component optional moodle component to sent xAPI tracking
     *
     * @return string The embedable code to display a H5P file.
     */
    public static function display(string $url, \stdClass $config, bool $preventredirect = true,
            string $component = ''): string {
        global $OUTPUT, $DB, $PAGE, $USER;

        $h5pid = '';
        $attempts = null;

        if($PAGE->pagelayout == 'incourse' && $PAGE->pagetype == 'mod-h5pactivity-view') {
            $sql = "SELECT instance FROM {course_modules} WHERE id=" . $PAGE->cm->id;
            $h5pid = $DB->get_record_sql($sql)->instance;
            // Get attempts
            $sql = "SELECT MAX(attempt) as attempts FROM {h5pactivity_attempts} WHERE userid = " . $USER->id . " AND h5pactivityid=". $h5pid;
            $attempts = $DB->get_record_sql($sql)->attempts;
            if ($attempts == null) {$attempts = '0';}
        }

        $params = [
                'url' => $url,
                'preventredirect' => $preventredirect,
                'component' => $component,
            ];

        $optparams = ['frame', 'export', 'embed', 'copyright'];
        foreach ($optparams as $optparam) {
            if (!empty($config->$optparam)) {
                $params[$optparam] = $config->$optparam;
            }
        }
        $fileurl = new \moodle_url('/h5p/embed.php', $params);

        $template = new \stdClass();
        $template->embedurl = $fileurl->out(false);
        $template->h5pid = $h5pid;
        $template->attempts = $attempts;

        $result = $OUTPUT->render_from_template('core_h5p/h5pembed', $template);
        $result .= self::get_resize_code();
        return $result;
    }
```
# Copy and paste this code in h5p/template/h5piframe.mustache
```
{{!
    This file is part of Moodle - http://moodle.org/

    Moodle is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    Moodle is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with Moodle.  If not, see <http://www.gnu.org/licenses/>.
}}
{{!
    @template core_h5p/h5piframe

    This template will render an iframe for h5p content.

    Variables required for this template:
    * h5pid - The database id for the H5P content

    Example context (json):
    {
        "h5pid": 123
    }

}}
<div class="h5p-iframe-wrapper">
    <iframe id="h5p-iframe-{{h5pid}}" class="h5p-iframe" data-content-id="{{h5pid}}"
        style="height:1px; min-width: 100%" src="about:blank">
    </iframe>
</div>

{{#js}}
    var cssLink = document.createElement("link");
    cssLink.href = "{{cssurl}}";
    if(cssLink.href) {
        cssLink.rel = "stylesheet";
        cssLink.type = "text/css";
        $('#h5p-iframe-{{h5pid}}').contents().find("head").append(cssLink);
    }
{{/js}}
```
# Copy and paste this code in h5p/template/h5pembed.mustache
```
# Copy and paste this code in h5p/template/h5pembed.mustache
{{!
    This file is part of Moodle - http://moodle.org/

    Moodle is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    Moodle is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with Moodle.  If not, see <http://www.gnu.org/licenses/>.
}}
{{!
    @template core_h5p/h5pembed

    This template will render the embed code shown in the H5P content embed popup.

    Variables required for this template:
    * embedurl - The URL with the H5P file to embed

    Example context (json):
    {
        "embedurl": "http://example.com/h5p/embed.php?url=testurl"
    }

}}
<iframe src="{{embedurl}}&attempts={{attempts}}" name="h5player" width=":w" height=":h"
   allowfullscreen="allowfullscreen" class="h5p-player w-100 border-0"
   style="min-height: 230px;">
</iframe>
```
