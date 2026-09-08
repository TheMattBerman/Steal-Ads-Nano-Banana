# Creative-generator export cleanup

The generator now goes from Parse VisionStruct Response directly to Build Image Request. The second Claude image-spec construction request and its two preparation/parsing nodes were removed because image generation already uses generated_visionstruct.

Both success and copy-only Sheet writes keep the existing image_prompt_json column. Its value is now the exact provider request (model, messages, and token limit) used for the attempted image, rather than a separately generated layout spec. layer_count is cleared because the provider request has no layout_layers contract. The actual prompt remains available as image_prompt_text in execution data and messages[0].content in the saved request.

Before importing this export into a running workflow, update any Sheet consumers that expect image_prompt_json.layout_layers or use layer_count. Historical rows keep their old JSON shape. No live workflow was changed or executed by this cleanup.
