# evm-tests-suite-parsed

A collection of all the tests of the Ethereum official test suite, with Shanghai Hard Fork consensus rules, in serialized form to be fed to the plonky2 zkEVM.

The JSON files consist in the `GenerationInputs` objets that are passed to the plonky2 zkEVM prover to generate proofs of Ethereum state transition.

It is currently supporting plonky2 commit revision 5572da30d7ab818594cf8659839fa832dfcf1d3d and onwards.

## Usage

These serialized test files can be read as such:

```rust
    let bytes = std::fs::read("serialized_test_file.json").unwrap();
    let inputs: GenerationInputs = serde_json::from_slice(&bytes).unwrap();
```

The deserialized `GenerationInputs` can be fed to the prover to test its correctness, as follows:

```rust
type F = plonky2::field::goldilocks_field::GoldilocksField;
const D: usize = 2;
type C = plonky2::plonk::config::KeccakGoldilocksConfig;

fn main() -> anyhow::Result<()> {
    // Set the proving configuration.
    let all_stark = AllStark::<F, D>::default();
    let config = StarkConfig::standard_fast_config();

    // Deserialize the test file.
    let bytes = std::fs::read("serialized_test_file.json").unwrap();
    let inputs: GenerationInputs = serde_json::from_slice(&bytes).unwrap();

    // Generate the proof for the deserialized inputs and verify it.
    let mut timing = TimingTree::new("prove", log::Level::Debug);
    let proof = prove::<F, C, D>(&all_stark, &config, inputs, &mut timing)?;
    verify_proof(&all_stark, proof, &config)
}
```