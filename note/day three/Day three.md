推荐的步骤：

1. [Github Classroom方式进行在线OS 环境配置](https://learningos.github.io/rust-based-os-comp2022/0setup-devel-env.html#github-classroomos)中的：1->2->3->4
2. [Rust 开发环境配置](https://learningos.github.io/rust-based-os-comp2022/0setup-devel-env.html#rust)
   - 通过`rustc --version`你会发现rust应该处于nightly版本，请不要切换为stable版本，否则无法中[试运行 rCore-Tutorial](https://learningos.github.io/rust-based-os-comp2022/0setup-devel-env.html#rcore-tutorial)中执行`LOG=DEBUG make run`
   - 若切换为 stable 版本，请使用 `rustup override set nightly` 与 `rustup default nightly`切换回nightly版本
3. [Qemu 模拟器安装](https://learningos.github.io/rust-based-os-comp2022/0setup-devel-env.html#qemu)
4. 此时，你的控制台显示的位置应该在`/workspaces/lab0-0-setup-env-run-os1-你的github名`
   - 如果你是使用github的 codespace，请不要执行[试运行 rCore-Tutorial](https://learningos.github.io/rust-based-os-comp2022/0setup-devel-env.html#rcore-tutorial)中的`git clone https://github.com/LearningOS/rust-based-os-comp2022.git`，因为你已经clone到codeplace环境
   - 可以直接执行 `make setupclassroom`（完成后，你的项目已提交）
5. 继续执行 `cd os1` 与 `LOG=DEBUG make run`，试运行 rCore-Tutorial

第0章完成