# Exports the Gemma constraint provider stub source so it can be referenced
# as a Bazel label from runtime/components/constrained_decoding/BUILD on iOS.
# The stub defines LiteRtLmGemmaModelConstraintProvider_{Create,Destroy,
# CreateConstraintFromTools} as no-op functions (returns nullptr, prints
# warning). This unblocks the iOS xcframework link while grammar-constrained
# decoding remains functionally disabled on iOS.
exports_files(["gemma_model_constraint_provider.cc"])
