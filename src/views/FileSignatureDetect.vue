<template>
    <my-page title="文件头信息检测">
        <div class="search-bar">
           <input type="file" id="files" name="files[]" multiple/>
            <output id="list"></output>
            <!-- <input type="file" @change="fileChange($event)"/> -->
            <div v-if="result.finish">
                <div>文件头：{{ result.coding }} ...</div>
                <div>格式：{{ result.ext }}</div>
            </div>
        </div>
    </my-page>
</template>

<script>
    /* eslint-disable */
    export default {
        data () {
            return {
                result: {
                    finish: true,
                    coding: '',
                    ext: ''
                }
            }
        },
        computed: {
        },
        mounted() {
            console.log((255).toString(16))
            let _this = this
            var fileSignatures = {
                'mp3': [
                    // MPEG-1 Layer 3 file without an ID3 tag or with an ID3v1 tag (which's appended at the end of the file)
                    [0xFF, 0xFB],
                    // MP3 file with an ID3v2 container
                    [0x49, 0x44, 0x33]
                ],
                'wav': [
                    // Waveform Audio File Format
                    // Empty slots can be any byte. Can't look at only first 4 or else .avi files match
                    [0x52, 0x49, 0x46, 0x46, , , , , 0x57, 0x41, 0x56, 0x45]
                ],
                'png': [
                    //89 50 4E 47 0D 0A 1A 0A
                    [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]
                ],
            };

            /**
             * Compare an Uint8Array with an expected file signature.
             * Can't do a direct equality check since some signatures (e.gfor wav files) have wildcard slots.
             * @param {array} sig - pattern from fileSignatures
             * @param {Uint8Array} actual - bytes from file (should already be sliced to match length of sig)
             * @returns {boolean} - do they match
             */
            var compareSignature = function(sig, actual) {
                if (sig.length !== actual.length) return false;
                for (var i = 0, l = sig.length; i < l; i++) {
                    if (sig[i] !== actual[i] && typeof sig[i] !== 'undefined') return false;
                }
                return true;
            };

            /**
             * @param {Uint8Array} uint8
             * @param {string} type
             * @returns {boolean}
             */
            var matchesFileType = function(uint8, type) {
                for (let item of uint8) {
                    console.log(item.toString(16))
                }
                return _.find(fileSignatures[type], function(sig) {
                    return compareSignature(sig, uint8.subarray(0, sig.length));
                });
            };

            /**
             * Detect, through file signature / mime sniffing detection, if a given File
             * matches an expected type or types. The types supported are the keys in
             * fileSignatures above.
             * @param {File} file
             * @param {(string|string[])} types - e.g. 'mp3' or ['mp3', 'wav']
             * @param {function} - callback which is passed a boolean
             */
            var verifyFileType = function(file, types, cb) {
                if (_.isString(types)) types = [types];

                // Calculate the longest file signature for any of the requested
                // types, so we know how many bytes of this file to look at.
                var bytesNeeded = types.reduce(function(prevMax, type, idx, arr) {
                    var sigs = fileSignatures[type];
                    return Math.max.apply(this, [prevMax].concat(sigs.map(function(sig) {
                        return sig.length;
                    })));
                }, 0);
                console.log(bytesNeeded)

                // Load file into ArrayBuffer and see if its first few bytes match
                // the signature of any of our requested types. Let callback know.
                var reader = new FileReader();
                reader.onload = function(e) {
                    // Load only as many bytes from the array buffer as necessary
                    var arrayBuffer = e.currentTarget.result;
                    var bytes = new Uint8Array(arrayBuffer, 0, bytesNeeded);
                    console.log(bytes)
                    var match = _.find(types, function(type) {
                        console.log('匹配' + type)
                        return matchesFileType(bytes, type);
                    });
                    cb(match);
                };
                reader.readAsArrayBuffer(file);
            };

            // Expose public interface
            var FileDetector = {
                verifyFileType: verifyFileType
            };

            function handleFileSelect(evt) {
                var files = evt.target.files; // FileList object
                let file = files[0]
                let reader = new FileReader()
                reader.onload = e => {
                    var arrayBuffer = e.target.result
                    var bytes = new Uint8Array(arrayBuffer, 0, 8)
                    console.log('哈哈')
                    console.log(bytes)
                    let arr = []
                    for (let item of bytes) {
                        arr.push(item.toString(16))
                    }
                    _this.result.coding = arr.join(' ')
                };
                reader.readAsArrayBuffer(file)

                // files is a FileList of File objects. List some properties.
                // var output = [];
                // for (var i = 0, f; f = files[i]; i++) {
                //     FileDetector.verifyFileType(f, ['png'], function(valid) {
                //         console.log('Valid mp3 or wave file: ' + !!valid)
                //     });
                   
                // }
            }
            document.getElementById('files').addEventListener('change', handleFileSelect, false);
        },
        methods: {
        }
    }
</script>

<style lang="scss" scoped>
    
</style>
