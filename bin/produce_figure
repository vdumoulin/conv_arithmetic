#!/usr/bin/env python
import argparse
import itertools
import os
import subprocess
from glob import glob

import numpy
import six

numpy.random.seed(1234)


def make_numerical_tex_string(step, input_size, output_size, padding,
                              kernel_size, stride, dilation, mode):
    """Creates a LaTeX string for a numerical convolution animation.

    Parameters
    ----------
    step : int
        Which step of the animation to generate the LaTeX string for.
    input_size : int
        Convolution input size.
    output_size : int
        Convolution output size.
    padding : int
        Zero padding.
    kernel_size : int
        Convolution kernel size.
    stride : int
        Convolution stride.
    mode : str
        Kernel mode, one of {'convolution', 'average', 'max'}.

    Returns
    -------
    tex_string : str
        A string to be compiled by LaTeX to produce one step of the
        animation.

    """
    if mode not in ('convolution', 'average', 'max'):
        raise ValueError("wrong convolution mode, choices are 'convolution', "
                         "'average' or 'max'")
    if dilation != 1:
        raise ValueError("Only a dilation of 1 is currently supported for numerical output")
    max_steps = output_size ** 2
    if step >= max_steps:
        raise ValueError('step {} out of bounds (there are '.format(step) +
                         '{} steps for this animation'.format(max_steps))

    with open(os.path.join('templates', 'numerical_figure.txt'), 'r') as f:
        tex_template = f.read()

    total_input_size = input_size + 2 * padding

    input_ = numpy.zeros((total_input_size, total_input_size), dtype='int32')
    input_[padding: padding + input_size,
           padding: padding + input_size] = numpy.random.randint(
        low=0, high=4, size=(input_size, input_size))
    kernel = numpy.random.randint(
        low=0, high=3, size=(kernel_size, kernel_size))
    output = numpy.empty((output_size, output_size), dtype='float32')
    for offset_x, offset_y in itertools.product(range(output_size),
                                                range(output_size)):
        if mode == 'convolution':
            output[offset_x, offset_y] = (
                input_[stride * offset_x: stride * offset_x + kernel_size,
                       stride * offset_y: stride * offset_y + kernel_size] * kernel).sum()
        elif mode == 'average':
            output[offset_x, offset_y] = (
                input_[stride * offset_x: stride * offset_x + kernel_size,
                       stride * offset_y: stride * offset_y + kernel_size]).mean()
        else:
            output[offset_x, offset_y] = (
                input_[stride * offset_x: stride * offset_x + kernel_size,
                       stride * offset_y: stride * offset_y + kernel_size]).max()

    offsets = list(itertools.product(range(output_size - 1, -1, -1),
                                     range(output_size)))
    offset_y, offset_x = offsets[step]

    if mode == 'convolution':
        kernel_values_string = ''.join(
            "\\node (node) at ({0},{1}) {{\\tiny {2}}};\n".format(
                i + 0.8 + stride * offset_x, j + 0.2 + stride * offset_y,
                kernel[kernel_size - 1 - j, i])
            for i, j in itertools.product(range(kernel_size),
                                          range(kernel_size)))
    else:
        kernel_values_string = '\n'

    return six.b(tex_template.format(**{
        'PADDING_TO': '{0},{0}'.format(total_input_size),
        'INPUT_FROM': '{0},{0}'.format(padding),
        'INPUT_TO': '{0},{0}'.format(padding + input_size),
        'INPUT_VALUES': ''.join(
            "\\node (node) at ({0},{1}) {{\\footnotesize {2}}};\n".format(
                i + 0.5, j + 0.5, input_[total_input_size - 1 - j, i])
            for i, j in itertools.product(range(total_input_size),
                                          range(total_input_size))),
        'INPUT_GRID_FROM': '{},{}'.format(stride * offset_x,
                                          stride * offset_y),
        'INPUT_GRID_TO': '{},{}'.format(stride * offset_x + kernel_size,
                                        stride * offset_y + kernel_size),
        'KERNEL_VALUES': kernel_values_string,
        'OUTPUT_TO': '{0},{0}'.format(output_size),
        'OUTPUT_GRID_FROM': '{},{}'.format(offset_x, offset_y),
        'OUTPUT_GRID_TO': '{},{}'.format(offset_x + 1, offset_y + 1),
        'OUTPUT_VALUES': ''.join(
            "\\node (node) at ({0},{1}) {{\\tiny {2:.1f}}};\n".format(
                i + 0.5, j + 0.5, output[output_size - 1 - j, i])
            for i, j in itertools.product(range(output_size),
                                          range(output_size))),
        'XSHIFT': '{}cm'.format(total_input_size + 1),
        'YSHIFT': '{}cm'.format((total_input_size - output_size) // 2),
    }))


def make_arithmetic_tex_string(step, input_size, output_size, padding,
                               kernel_size, stride, dilation, transposed):
    """Creates a LaTeX string for a convolution arithmetic animation.

    Parameters
    ----------
    step : int
        Which step of the animation to generate the LaTeX string for.
    input_size : int
        Convolution input size.
    output_size : int
        Convolution output size.
    padding : int
        Zero padding.
    kernel_size : int
        Convolution kernel size.
    stride : int
        Convolution stride.
    dilation: int
        Input Dilation
    transposed : bool
        If ``True``, generate strings for the transposed convolution
        animation.

    Returns
    -------
    tex_string : str
        A string to be compiled by LaTeX to produce one step of the
        animation.

    """
    kernel_size = (kernel_size - 1)*dilation + 1
    if transposed:
        # Used to add bottom-padding to account for odd shapes
        bottom_pad = (input_size + 2 * padding - kernel_size) % stride

        input_size, output_size, padding, spacing, stride = (
            output_size, input_size, kernel_size - 1 - padding, stride, 1)
        total_input_size = output_size + kernel_size - 1
        y_adjustment = 0
    else:
        # Not used in convolutions
        bottom_pad = 0

        spacing = 1
        total_input_size = input_size + 2 * padding
        y_adjustment = (total_input_size - (kernel_size - stride)) % stride

    max_steps = output_size ** 2
    if step >= max_steps:
        raise ValueError('step {} out of bounds (there are '.format(step) +
                         '{} steps for this animation'.format(max_steps))

    with open(os.path.join('templates', 'arithmetic_figure.txt'), 'r') as f:
        tex_template = f.read()
    with open(os.path.join('templates', 'unit.txt'), 'r') as f:
        unit_template = f.read()

    offsets = list(itertools.product(range(output_size - 1, -1, -1),
                                     range(output_size)))
    offset_y, offset_x = offsets[step]

    return six.b(tex_template.format(**{
        'PADDING_TO': '{0},{0}'.format(total_input_size),
        'INPUT_UNITS': ''.join(
            unit_template.format(padding + spacing * i,
                                 bottom_pad + padding + spacing * j,
                                 padding + spacing * i + 1,
                                 bottom_pad + padding + spacing * j + 1)
            for i, j in itertools.product(range(input_size),
                                          range(input_size))),
        'INPUT_GRID_FROM_X': '{}'.format(
            stride * offset_x),
        'INPUT_GRID_FROM_Y': '{}'.format(
            y_adjustment + stride * offset_y),
        'INPUT_GRID_TO_X': '{}'.format(
            stride * offset_x + kernel_size),
        'INPUT_GRID_TO_Y': '{}'.format(
            y_adjustment + stride * offset_y + kernel_size),
        'DILATION': '{}'.format(dilation),
        'OUTPUT_BOTTOM_LEFT': '{},{}'.format(offset_x, offset_y),
        'OUTPUT_BOTTOM_RIGHT': '{},{}'.format(offset_x + 1, offset_y),
        'OUTPUT_TOP_LEFT': '{},{}'.format(offset_x, offset_y + 1),
        'OUTPUT_TOP_RIGHT': '{},{}'.format(offset_x + 1, offset_y + 1),
        'OUTPUT_TO': '{0},{0}'.format(output_size),
        'OUTPUT_GRID_FROM': '{},{}'.format(offset_x, offset_y),
        'OUTPUT_GRID_TO': '{},{}'.format(offset_x + 1, offset_y + 1),
        'OUTPUT_ELEVATION': '{}cm'.format(total_input_size + 1),
    }))


def compile_figure(which_, name, step, **kwargs):
    if which_ == 'arithmetic':
        tex_string = make_arithmetic_tex_string(step, **kwargs)
    else:
        tex_string = make_numerical_tex_string(step, **kwargs)
    jobname = '{}_{:02d}'.format(name, step)
    p = subprocess.Popen(['pdflatex', '-jobname={}'.format(jobname),
                          '-output-directory', 'pdf'],
                         stdin=subprocess.PIPE, stdout=subprocess.PIPE,
                         stderr=subprocess.PIPE)
    stdoutdata, stderrdata = p.communicate(input=tex_string)
    # Remove logs and aux if compilation was successfull
    if '! LaTeX Error' in stdoutdata or '! Emergency stop' in stdoutdata:
        print('! LaTeX Error: check the log file in pdf/{}.log'.format(jobname))
    else:
        subprocess.call(['rm'] + glob('pdf/{}.aux'.format(jobname)) +
                        glob('pdf/{}.log'.format(jobname)))


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Compile a LaTeX figure as part of a convolution "
                    "animation.")

    subparsers = parser.add_subparsers()

    parent_parser = argparse.ArgumentParser(add_help=False)
    parent_parser.add_argument("name", type=str, help="name for the animation")
    parent_parser.add_argument("step", type=int, help="animation step")
    parent_parser.add_argument("-i", "--input-size", type=int, default=5,
                               help="input size")
    parent_parser.add_argument("-o", "--output-size", type=int, default=3,
                               help="output size")
    parent_parser.add_argument("-p", "--padding", type=int, default=0,
                               help="zero padding")
    parent_parser.add_argument("-k", "--kernel-size", type=int, default=3,
                               help="kernel size")
    parent_parser.add_argument("-s", "--stride", type=int, default=1,
                               help="stride")
    parent_parser.add_argument("-d", "--dilation", type=int, default=1,
                               help="dilation")

    subparser = subparsers.add_parser('arithmetic', parents=[parent_parser],
                                      help='convolution arithmetic animation')
    subparser.add_argument("--transposed", action="store_true",
                           help="animate a transposed convolution")
    subparser.set_defaults(which_='arithmetic')

    subparser = subparsers.add_parser('numerical', parents=[parent_parser],
                                      help='numerical convolution animation')
    subparser.add_argument("-m", "--mode", type=str, default='convolution',
                           choices=('convolution', 'average', 'max'),
                           help="kernel mode")
    subparser.set_defaults(which_='numerical')

    args = parser.parse_args()
    args_dict = vars(args)
    which_ = args_dict.pop('which_')
    name = args_dict.pop('name')
    step = args_dict.pop('step')

    compile_figure(which_, name, step, **args_dict)
